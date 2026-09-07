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
