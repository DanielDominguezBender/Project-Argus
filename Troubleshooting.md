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

---

## Proxmox installation failures caused by unreliable USB media

### Symptoms

Several Proxmox installation attempts failed with different errors.

Proxmox VE 9.2:

`installation of package vncterm_1.9.2_amd64.deb failed`

Proxmox VE 8.4:

`unsquashfs ... pve-base.squashfs failed`

### Investigation

- ISO SHA256 verified.
- Installation media recreated with balenaEtcher.
- Installation media recreated with `dd`.
- Graphical installer tested.
- Terminal installer tested.
- Kernel compatibility option tested.
- Same USB device continued producing installation failures.

### Root Cause

Unreliable Kingston USB installation media.

### Resolution

A different 4 GB USB drive was used to create the Proxmox installer.

Proxmox VE 8.4 installed successfully on the first attempt.

### Lesson Learned

When installation errors appear inconsistent or occur while extracting packages, verify not only the ISO checksum but also the physical installation media.

---

09.09.2026

## SSH connectivity failure after DHCP reservation changes

### Context

Project Argus uses DHCP reservations on the TP-Link/Deco network so that each Proxmox VM keeps a stable IP address.

Planned addressing:

- `linux01` → `192.168.68.51`
- `linux02` → `192.168.68.52`
- `docker01` → `192.168.68.53`
- `pve01` → `192.168.68.200`

After creating the DHCP reservations, SSH connectivity to `linux01` and `linux02` stopped working correctly.

---

### Symptoms

#### linux02

SSH returned:

```bash
ssh: connect to host 192.168.68.52 port 22: Connection refused
```

Inside the VM, the SSH service was initially not enabled for automatic startup.

After starting it:
```bash
sudo systemctl start ssh
```

the service showed:

```
Active: active (running)
Server listening on 0.0.0.0 port 22
Server listening on :: port 22
```

The service was then permanently enabled:

```
sudo systemctl enable ssh
```

Connectivity was restored.

## linux01

linux01 had an active network interface:

```
ens18: UP, LOWER_UP
```

but initially had no usable IPv4 address.

nmcli device show ens18 showed IPv6 configuration but no IPv4 gateway or IPv4 address.

The system was physically connected to the network, but the DHCP reservation had not yet been correctly applied to the active NetworkManager connection.

## Investigation

The following checks were performed:

```
ip addr
ip route
systemctl status NetworkManager
nmcli device status
nmcli device show ens18
```

NetworkManager was active and the interface was connected.

The DHCP method was verified as:

```
ipv4.method: auto
```

The TP-Link/Deco reservation was also checked against the VM MAC address.

For linux01:

```
MAC: BC:24:11:6D:5A:E9
Reserved IP: 192.168.68.51
```

ARP checks from the management workstation were used to verify IP-to-MAC ownership:

```
arp -an | grep 192.168.68.51
arp -an | grep 192.168.68.52
```

This helped confirm that the expected IP addresses were associated with the correct virtual NICs.

## Resolution

The NetworkManager connection profile on linux01 was explicitly restarted:

```
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

After reconnecting the profile, DHCP was renegotiated and the reserved IPv4 address was correctly assigned:

```
inet 192.168.68.51/22
```

The default gateway was also restored:

```
default via 192.168.68.1
```

SSH connectivity then worked again:

```
ssh linux01
```

## Why nmcli connection down/up worked

The command:

```
sudo nmcli device disconnect ens18
```

operates mainly on the physical/logical network device state.

In contrast:

```
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

reapplies the complete NetworkManager connection profile, including:

```
IPv4 configuration
DHCP negotiation
routes
DNS
profile-to-interface association
```

This forced a fresh DHCP negotiation with the TP-Link/Deco server and caused the newly configured reservation to be applied.

## Root Cause

The issue was not primarily SSH.

The root cause was stale/incomplete network state after changing DHCP reservations.

linux01 had:

-link state available
-NetworkManager active
-SSH service available

but did not yet have the expected reserved IPv4 address.

For linux02, the additional issue was that ssh.service was not enabled for automatic startup.

## Lessons Learned

Do not troubleshoot SSH before validating the host's actual IPv4 configuration.

Verify the full identity chain:

-hostname
-IP address
-MAC address
-ARP entry
-DHCP reservation

UP and LOWER_UP only confirm link state; they do not guarantee valid Layer 3 configuration.
After changing DHCP reservations, explicitly restarting the NetworkManager connection profile may be required.
Connection refused often means the target host is reachable but nothing is listening on the requested port.
Operation timed out usually points more toward reachability, filtering, or routing.
SSH host-key warnings should never be suppressed blindly; first verify that the IP still belongs to the expected machine.
