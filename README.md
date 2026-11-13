# S3 Storage Appliance

A single-VM HTTPS S3 target for VMware Data Services Manager lab backups. The clean image uses Ubuntu Server 24.04 Baseline 1.5, SeaweedFS 4.53, and NGINX. Each deployment generates its own credentials and TLS key.

## Download

[Download S3 Storage Appliance 1.0.3](https://github.com/raleyj/s3-storage/releases/tag/v1.0.3) with its SHA256 file, deployment guide, and manifest. The OVA is 1,075,681,280 bytes. SHA256:

```text
a468442d253347a5646a2990081d2a09622c8363959d8d955d12febaf8f907af  S3-Storage-Appliance-1.0.3.ova
```

Version 1.0.3 updates glibc to `2.39-0ubuntu8.12` using a refreshed, checksum-locked offline package bundle. The exact OVA passed vCenter import and NIC-disconnected first-boot and reboot tests: signed HTTPS S3 CRUD, multipart readback, bucket isolation, local health, and persistence of objects, identities, and TLS. The clean disk contains no generated deployment secrets. A DSM 9.1.1 system recovery passed on the earlier 1.0.2 OVA and was not repeated on 1.0.3.

See the [1.0.3 deployment guide](Deployment-Guide-1.0.3.md), [manifest](S3-Storage-Appliance-1.0.3.manifest.json), and [security notes](SECURITY-1.0.3.md). Earlier versions remain available for historical comparison.

This is a single-VM lab appliance without object-level redundancy. Keep an independent recovery copy. No deployment passwords, S3 credentials, TLS private keys, or backup data are embedded in the OVA.

The JTEC lab runs a separate 15-minute signed S3 and DSM backup/WAL age monitor on an always-on OpsChat systemd timer. It delivers alerts to the dedicated Mattermost S3 Alerts channel; new deployments need their own external monitoring.
