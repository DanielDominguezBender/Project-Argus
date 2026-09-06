# 06.09.2026

## Proxmox unreachable despite correct IP configuration

### Symptoms

- Proxmox configured with:
  - IP: `192.168.68.200/24`
  - Gateway: `192.168.68.1`
- `vmbr0` was `UP`
- Physical NIC showed `LOWER_UP`
- Ping to gateway failed:
  - `Destination Host Unreachable`
- Proxmox Web UI was inaccessible.

### Initial hypothesis

Possible issues with:

- Linux bridge `vmbr0`
- Proxmox firewall
- Physical NIC
- PLC connectivity

### Root cause

The Proxmox host was connected through a PLC whose source adapter was connected directly to the Movistar HGU.

The `192.168.68.0/24` network belongs to the TP-Link/Deco LAN, so the Proxmox host was physically connected to the wrong network segment.

### Resolution

Connect the Proxmox host, or the PLC source adapter, to the TP-Link/Deco LAN.

After reconnecting:

- Ping to `192.168.68.200` succeeded
- Gateway connectivity was restored
- Proxmox Web UI became available at:

`https://192.168.68.200:8006`

### Lesson Learned

When a network interface reports `UP` and `LOWER_UP` but the host cannot reach its configured gateway, verify the physical network topology and Layer 2/Layer 3 segment before modifying:

- bridge configuration
- firewall rules
- IP addressing
- routing
