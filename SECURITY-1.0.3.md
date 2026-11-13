# S3 Storage Appliance 1.0.3 security status

Version 1.0.3 uses Ubuntu Server 24.04 Baseline 1.5, kernel package `6.8.0-149.149`, glibc `2.39-0ubuntu8.12`, and pinned official SeaweedFS 4.53. The nftables IPv4-options rule remains enabled. A checksum-locked 90-package offline bundle supplies the first-boot dependencies without network access.

The November 13, 2026 Trivy OS scan of the exact 1.0.3 build disk reported 6,994 package matches, compared with 7,000 for 1.0.2 under the same scanner data. `libc6` matches fell from six to zero. Most remaining matches are associated with Ubuntu kernel packages for which the scan did not list a fixed version. Scanner matches require package and reachability review; they are not proof of exploitability.

SeaweedFS 4.53 was the latest official release when checked on November 13, 2026. Binary-mode `govulncheck` still reported six distinct Go advisories. Binary mode cannot prove that an affected request path is reachable in this appliance. Reassess when upstream publishes a new binary.

The exact 1.0.3 OVA passed a disconnected vCenter first boot and reboot with signed HTTPS S3 operations, multipart readback, bucket isolation, and persistent data and identities. A DSM 9.1.1 system recovery was tested with the earlier 1.0.2 OVA and was not repeated on 1.0.3. This remains a single-VM appliance without object-level redundancy; maintain independent recovery copies and external monitoring.

