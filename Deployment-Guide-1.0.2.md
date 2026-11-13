# S3 Storage Appliance

The appliance provides a reusable, single-VM HTTPS S3 target built from Ubuntu Server 24.04 Baseline 1.5 and SeaweedFS 4.53. NGINX terminates TLS. The image contains the checksum-pinned offline package bundle and initializes itself on first boot without downloading software. It is a lab storage appliance, with no object-level redundancy.

The current qualified local build is `release/S3-Storage-Appliance-1.0.2.ova`, accompanied by a SHA256 file and build manifest. It includes the nftables mitigation for CVE-2026-98432 and upgrades SeaweedFS from 4.48 to 4.53. Versions 1.0.0 and 1.0.1 remain available for historical comparison but should not be used for new deployments. Version 1.0.2 passed OVF Tool validation, vCenter's read-only import-spec validation, fresh DHCP and static first boot, signed HTTPS S3 operations, bucket isolation, reboot persistence, and the missing-data-disk guard. The exact published OVA passed vCenter guest runtime checks after first boot and reboot, and DSM 9.1.1 system recovery completed from a backup copied to a clean OVA instance. OS and Go binary scans are summarized in `evidence/vulnerability-summary-1.0.2.json` and `evidence/seaweedfs-vulnerability-summary-4.53.json`. Default resources are 4 vCPU, 8 GiB RAM, a 60 GiB OS disk and a separate 500 GiB data disk. Use thin provisioning. Increase the disks before first boot if needed; the secondary disk is formatted to its full initial size. Later data-disk expansion requires filesystem expansion.

## Deployment

1. Deploy the OVA in vSphere and map **VM Network** to the intended port group. Select the destination datastore and thin provisioning.
2. Set a unique hostname. Under **Ubuntu access**, supply **Password for default user (ubuntu)**. The field is masked, no password is embedded in the image, and there is no forced first-login password change. Leave **Enable SSH password authentication** checked for SSH password login, or uncheck it for SSH-key-only access. The deployment password still applies to the Ubuntu account when SSH password authentication is disabled. Supply an SSH public key if desired. Root remains locked. Supply at least a password or SSH public key so you can administer the VM. vCenter administrators can access deployment metadata, including supplied passwords.
3. Choose DHCP or Static. Static mode requires address, dotted-decimal netmask, gateway and DNS servers. NTP values are optional and apply in Static mode; DHCP mode uses Ubuntu's NTP defaults. Set **DNS search domain** for internal domains such as a unicast `.local` domain.
4. Enter **S3 endpoint DNS name** and create the corresponding DNS record. Blank uses the deployment hostname. Set the region and two bucket names, or retain `us-east-1`, `dsm-system` and `dsm-postgres`. The buckets must have different names.
5. Restrict **SSH administrator source CIDRs** and **HTTPS S3 client source CIDRs** to the appropriate networks. Defaults allow private IPv4 networks. These are comma-separated CIDR lists. SSH listens on port 22 and S3 HTTPS on port 443. Internal storage ports bind to loopback.
6. Power on and allow initialization to complete. `cloud-init status --wait` confirms Ubuntu initialization; S3 initialization follows it. Check `sudo systemctl status s3-appliance-initialize` and wait for **active (exited)**. Inspect `sudo journalctl -u s3-appliance-initialize --no-pager` if initialization fails.

First boot requires exactly one blank secondary disk, at least 10 GiB. Initialization refuses ambiguous disks or existing filesystem signatures. Never attach an existing data disk for automatic formatting. Do not clone a configured instance as a new deployment; import this clean OVA so identities and credentials are generated independently.

## Credentials and certificates

Log in as `ubuntu` with the supplied deployment password or SSH key. Run `sudo s3-appliance credentials` to display the two scoped service identities and endpoint settings. Treat that output as secret. The full server-side identity configuration, including the local administrative identity, is root/group restricted at `/etc/seaweedfs/s3.json`. S3 credentials are generated separately for each deployment and persist across reboots.

Each new deployment generates a unique 3072-bit RSA TLS key and self-signed server certificate, valid for one year and containing the configured endpoint DNS name in its SAN. Obtain the public certificate at `/etc/s3-appliance/tls/s3.crt` with sudo and explicitly trust it in test clients, or install a CA-issued server certificate before integration. Do not disable TLS verification.

Generate a CSR using the existing private key:

```sh
sudo s3-appliance csr > s3.csr
```

After CA enrollment, transfer the server certificate, corresponding key and issuer CA bundle securely to the VM. Install them using:

```sh
sudo s3-appliance install-tls --cert server-chain.pem --key server.key --ca issuer-ca.pem
```

Use a PEM server certificate with intermediates as needed. The command verifies hostname, trust chain and key match, tests NGINX configuration, reloads NGINX, and retains a protected previous copy for rollback. Clients must trust the issuing CA. No JTEC CA certificate or private key is included in the OVA.

## Client integration

Use `https://<configured S3 DNS name>`, the configured region, **force path style**, the appropriate bucket, and that bucket's service credentials. Anonymous requests and access to the other bucket are denied. Versioning, object lock and lifecycle expiration are not enabled by default. A browser returning **403** at the endpoint without credentials is expected.

For DSM, configure system and managed-database repositories separately. Add the issuing CA to DSM's supported S3 trust bundle and test actual backup and recovery for the installed DSM version. The clean 1.0.2 OVA completed a DSM 9.1.1 system recovery test in this lab. Test backup and recovery for your installed DSM version and configuration.

## Storage and operation

All SeaweedFS master, volume and filer state resides on `/srv/s3`. Services require the mount and bind to its mount unit; a missing data disk cannot redirect writes onto the OS disk. The services run under the dedicated `seaweedfs` account with restricted write paths. Volume size is limited to 1 GiB and bucket growth is configured one volume at a time.

Useful commands:

```sh
sudo s3-appliance status
sudo /usr/local/sbin/s3-appliance-health
sudo systemctl --failed
sudo journalctl -u seaweedfs-s3 --no-pager
sudo ufw status verbose
```

The health timer runs every fifteen minutes and checks the mount, services, free space, TLS expiry and anonymous-access denial. It writes local journal results; external alerting and client backup-age monitoring are not configured.

For a consistent recovery copy, stop NGINX and all four SeaweedFS services, copy the entire `/srv/s3` filesystem plus configuration, TLS material and credentials to independent storage, then restart and verify signed object reads. Protect the copied secrets. Volume files alone are insufficient. The OVA is a clean deployment template, not a backup of any deployed appliance.

## Build and qualification

The disk is customized offline from the checksum-verified, unbooted Ubuntu baseline. No running S3 deployment or test VM disk is used for release packaging. The image contains no S3 credentials, deployment passwords, TLS private keys, SSH host keys, OVF deployment seed or initialized application state. Tests use separate imports and credentials in a restricted directory outside the release folder.

Build sources are in `src/` and `scripts/`. `prepare.py` verifies source inputs and creates the payload. `build-final-disks.sh` runs under WSL as root using libguestfs/qemu-img to create a new offline OS disk. VMware vdiskmanager converts both disks to stream-optimized VMDKs; `package.py` builds the descriptor, manifest and OVA. Existing release files are not silently overwritten.

Qualification results are stored in `evidence/` and summarized in the release manifest. Version 1.0.2 runtime qualification used two fresh final-OVA VMware Workstation imports for DHCP and static networking. vCenter's read-only import-spec API validated all nineteen deployment fields, including the masked password field, and both disks. On November 12, 2026, the exact published 1.0.2 OVA was deployed in vCenter as a disposable VM. First boot and reboot passed signed HTTPS S3 CRUD, multipart, bucket isolation, and persisted identity/TLS checks. A disposable DSM 9.1.1 VM then completed system recovery from a backup copied to the clean OVA S3 target (17,697 objects; 14,809,405,641 bytes). The test VMs were shut down with NIC start disabled afterward.

The acceptance checks cover actual supplied-password account authentication, both SSH authentication choices, no forced password change, independent identities and secrets, signed S3/multipart operations, bucket isolation, the missing-data-disk startup guard, CSR/TLS helper commands, and persisted credentials and objects across reboot. The original running JTEC DSM backup VM is outside the qualification scope and is not used as a release disk. External NTP synchronization and managed PostgreSQL recovery against the clean OVA are not acceptance assertions for this generic image.

