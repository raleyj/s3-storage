# S3 Storage Appliance

A single-VM HTTPS S3 target for VMware Data Services Manager lab backups. The clean appliance image uses Ubuntu Server 24.04 Baseline 1.5, SeaweedFS 4.53, and NGINX. Each deployment generates its own credentials and TLS key.

## Download

[Download S3 Storage Appliance 1.0.2](https://github.com/raleyj/s3-storage/releases/tag/v1.0.2) with its SHA256 file, deployment guide, and manifest. The OVA is 858,152,960 bytes. SHA256:

```text
e06ec6989ace7ab78cebcb672602874c1aa240da1da7763dc151b6be2fa15c31  S3-Storage-Appliance-1.0.2.ova
```

Version 1.0.2 includes a network mitigation for CVE-2026-98432 and upgrades SeaweedFS from 4.48 to 4.53. The OVA passed OVF validation, vCenter import-spec validation, fresh DHCP and static first boots, signed HTTPS S3 operations, bucket isolation, reboot persistence, and the missing-data-disk guard. The exact published OVA also passed vCenter guest runtime checks after first boot and reboot. DSM 9.1.1 system recovery completed on a disposable DSM VM using a backup copied to this clean OVA as the S3 target. The running JTEC deployment separately passed an encrypted recovery-copy restore and DSM system and PostgreSQL restore drills.

See [the 1.0.2 deployment guide](Deployment-Guide-1.0.2.md), [manifest](S3-Storage-Appliance-1.0.2.manifest.json), and [security notes](SECURITY-1.0.2.md). Versions 1.0.0 and 1.0.1 remain available for historical comparison and should not be used for new deployments.

This is a single-VM lab appliance without object-level redundancy. Keep an independent recovery copy. No deployment passwords, S3 credentials, TLS private keys, or backup data are embedded in the OVA.
