# Changelog

## [0.1.0] - 2026-09-06

### Added

- Initial repository structure
- Project documentation
- Proxmox installation planning

---

## [0.2.0] - 2026-09-07

### Added

- Proxmox VE host on repurposed MacBook Pro
- Debian 13 VM (`linux01`)
- Ubuntu Server VM (`linux02`)
- SSH key authentication
- SSH aliases for both Linux hosts
- Ansible inventory
- First Ansible configuration-management playbook
- Thermal monitoring baseline
- Closed-lid headless operation
- Troubleshooting documentation

### Changed

- Configured systemd-logind to ignore lid-close events
- Added external cooling for the Proxmox host

### Fixed

- Proxmox installation failures caused by unreliable USB media
- Proxmox network connectivity caused by incorrect physical LAN placement
- Restored SSH connectivity after DHCP reservation changes.
- Corrected IPv4 lease application on `linux01` by restarting the NetworkManager connection profile.
- Enabled persistent SSH startup on `linux02`.

---

## [0.3.0] - 2026-09-11

### Added

- Cross-platform Project Guardian recovery test.
- Pi-hole redeployment from Raspberry Pi ARM64 to Proxmox AMD64.
- Stateful `/etc/pihole` backup and restore procedure.
- DNS and web-service validation after recovery.

### Validated

- Pi-hole Docker image multi-architecture support.
- Project Guardian Docker Compose portability.
- Persistent Pi-hole database and configuration recovery.
- Historical Pi-hole data successfully restored on `docker01`.