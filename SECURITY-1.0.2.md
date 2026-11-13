# Security and qualification notes for 1.0.2

- The appliance uses Ubuntu Server 24.04 Baseline 1.5 and SeaweedFS 4.53.
- The image includes a verified nftables mitigation for CVE-2026-98432. The Ubuntu 24.04 kernel in this build does not include a fixed package for that CVE.
- The Trivy rootfs scan recorded 52 unique non-kernel CVEs (57 medium and 42 low package matches), with no fixed package version available in the scan. Kernel matches were numerous and require applicability analysis; the scan alone does not establish exploitability.
- Binary-mode govulncheck found five Go advisories in the SeaweedFS binary, including three with newer module fixes. Binary-mode matches do not establish whether an S3 request can reach a vulnerable function.
- The OVA passed local DHCP and static first-boot tests, signed HTTPS S3 read/write and multipart tests, bucket isolation, reboot persistence, and the missing-data-disk guard. Its OVF was validated and vCenter accepted its import specification. The exact 1.0.2 OVA was imported to vCenter, but guest runtime validation was not completed.
- The running JTEC S3 appliance is a separate deployment. Its independent encrypted copy, S3 restore drill, DSM PostgreSQL restore/PITR, and DSM management-appliance restore passed on November 12, 2026. These recovery drills do not demonstrate that the clean 1.0.2 OVA itself was used as the recovery target.

The image is a lab release with no object-level redundancy. Limit client access to trusted networks, use a CA-issued certificate or explicitly trust the generated certificate, and retain an independent encrypted recovery copy.
