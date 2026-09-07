# Headless Operation

The Proxmox host is a repurposed MacBook Pro running with the lid closed in a vertical stand.

By default, closing the lid caused the host to suspend and lose network connectivity.

A systemd-logind override was created:

`/etc/systemd/logind.conf.d/99-lid-switch.conf`

Configuration:

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore

---

## Validation:

```
systemd-analyze cat-config systemd/logind.conf | grep HandleLidSwitch
``` 

The Proxmox host now remains online with the lid closed.

---
   
## Thermal monitoring

```     
docs/operations/thermal-monitoring.md

