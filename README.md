# Project Argus

Project Argus is a hands-on infrastructure operations homelab built around Proxmox VE, Linux virtualization, Ansible automation, Docker and infrastructure monitoring.

The project uses a repurposed MacBook Pro as a dedicated Proxmox host and focuses on building, operating, automating and troubleshooting a small virtual infrastructure environment.

The goal is not only to deploy services, but to understand how the different infrastructure layers interact:

- physical networking
- virtualization
- Linux administration
- configuration management
- containerization
- monitoring
- troubleshooting
- infrastructure reproducibility

---

## Architecture

```text
                         Mac Studio
                       Ansible Control
                             Node
                              │
                         SSH / Keys
                              │
                              ▼
                     ┌─────────────────┐
                     │      pve01      │
                     │   Proxmox VE    │
                     │  MacBook Pro    │
                     └────────┬────────┘
                              │
                            vmbr0
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
          linux01          linux02          docker01
          Debian 13        Ubuntu           Debian 13
              │               │                │
              │               │                └── Docker
              │               │                     │
              └────── Ansible managed ──────────────┘
```

## Current Infrastructure

Host	Role	Operating System	IP
pve01	Proxmox hypervisor	Proxmox VE 8.4	192.168.68.200
linux01	Linux managed node	Debian 13	192.168.68.51
linux02	Linux managed node	Ubuntu Server	192.168.68.52
docker01	Docker host	Debian 13	192.168.68.53

IP addresses are assigned using DHCP reservations based on VM MAC addresses.

---

## Hardware

Proxmox Host

Repurposed MacBook Pro Late 2012:

Intel x86-64 CPU
16 GB RAM
SSD storage
Gigabit Ethernet
Closed-lid headless operation
Vertical mounting
External cooling using a Mars Gaming MNBC2 laptop cooler

---

## Technology Stack

### Virtualization

Proxmox VE
KVM
VirtIO
Linux Bridge (vmbr0)
LVM-thin storage

### Operating Systems

Debian 13
Ubuntu Server

### Automation

Ansible Core 2.21
SSH key authentication
Ansible inventories
Playbooks
Idempotent configuration management

### Containers

Docker Engine
Docker Compose
Docker Buildx

### Planned Monitoring

Icinga
Prometheus
Grafana
Node Exporter

---

## Project Goals

Project Argus is designed to practice infrastructure engineering concepts in a realistic environment.

Key objectives include:

Build and manage virtual machines using Proxmox.
Understand virtual networking and Linux bridges.
Manage Linux systems remotely using SSH.
Automate system configuration using Ansible.
Deploy Docker hosts using Infrastructure Automation.
Validate configuration-management idempotency.
Monitor infrastructure resources and services.
Practice infrastructure troubleshooting.
Automate TLS certificate deployment.
Perform backup and disaster-recovery exercises.
Integrate selected workloads from Project Guardian.

---

## Current Milestones

Proxmox
 Proxmox installed on repurposed MacBook Pro
 Proxmox Web UI accessible
 Static management address configured
 Linux bridge vmbr0 validated
 Closed-lid headless operation configured
 Host thermal baseline captured

Virtual Machines
 linux01 deployed with Debian 13
 linux02 deployed with Ubuntu Server
 docker01 deployed with Debian 13
 QEMU Guest Agent configured
 DHCP reservations configured

SSH
 OpenSSH configured
 SSH key authentication
 Local SSH aliases

Example:

```bash
ssh linux01
ssh linux02
ssh docker01
```

Ansible
 Mac Studio configured as Ansible control node
 Linux inventory created
 Ansible connectivity validated
 Common packages deployed automatically
 Docker installation automated
 Playbook idempotency validated

Docker
 Docker Engine installed using Ansible
 Docker Compose plugin installed
 Docker Buildx installed
 Docker service enabled
 Non-root Docker access configured
 hello-world container successfully executed

---

## Ansible

The Mac Studio acts as the Ansible control node.

Inventory:

```INI
[linux_servers]
linux01
linux02
docker01

[docker_servers]
docker01
```

Connectivity can be validated with:

```bash
ansible linux_servers \
  -i ansible/inventory/hosts.ini \
  -m ping
```

A successful result confirms:

SSH connectivity
SSH authentication
remote Python availability
Ansible execution capability

---

## First Configuration Management Test

The first Ansible playbook standardized several common administration tools:

-curl
-git
-htop
-tree

Initial execution:

```bash
linux01 changed=1
linux02 changed=1
```

Second execution:

```bash
linux01 changed=0
linux02 changed=0
```

This demonstrates Ansible idempotency: once the desired state is reached, subsequent executions do not perform unnecessary changes.

---

## Docker Automation

Docker installation on docker01 is performed using Ansible.

The playbook:

installs required packages
creates the Docker keyring
downloads the Docker signing key
configures the Docker DEB822 repository
installs Docker Engine
installs Docker Compose
installs Docker Buildx
enables the Docker service
adds the user to the Docker group

The deployment was validated using:

```bash
docker --version
docker compose version
docker run hello-world
```

docker --version
docker compose version
docker run hello-world

A second Ansible execution returned:

```bash
changed=0
```

confirming the Docker deployment playbook is idempotent.

---

## Headless Proxmox Operation

The Proxmox host is operated with the laptop lid closed.

A systemd-logind override prevents the host from entering suspend mode:

```bash
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Validation:

```bash
systemd-analyze cat-config systemd/logind.conf \
  | grep HandleLidSwitch
```

The host remains reachable through Ethernet with the lid closed.

---

## Thermal Monitoring

Initial measurements were collected using lm-sensors.

### Without External Cooling
Metric	Value
CPU Package	~60°C
Core 0	~60°C
Core 1	~57°C
Internal fan	~2000 RPM
Battery	~34.5°C

### With Mars Gaming MNBC2
Metric	Value
CPU Package	~58°C
Core 0	~58°C
Core 1	~56°C
Internal fan	~2000 RPM
Battery	~33.5°C

The preliminary test showed a temperature reduction of approximately 1–3°C while the internal fan remained near its minimum speed.

These measurements will later be integrated into centralized monitoring.

---

## Troubleshooting Highlights

Project Argus intentionally documents failures and troubleshooting, not only successful deployments.

Examples include:

### Proxmox Installation Failure

Multiple Proxmox installation attempts failed with package and SquashFS errors.

Root cause:

Unreliable USB installation media.

Resolution:

A different USB device was used and Proxmox installed successfully.

---

## Incorrect Network Segment

Proxmox initially could not reach its configured gateway despite:

interface UP
interface LOWER_UP
correct vmbr0 configuration

Root cause:

The host was connected through a PLC attached to the ISP router instead of the TP-Link/Deco LAN.

Resolution:

The Ethernet path was moved behind the TP-Link/Deco network.

---

## SSH Failures After DHCP Reservation Changes

SSH connectivity failed after introducing DHCP reservations.

Troubleshooting included:

verifying SSH service status
checking IPv4 assignment
verifying MAC addresses
inspecting ARP entries
checking DHCP reservations
inspecting NetworkManager state

linux01 had link connectivity but no valid reserved IPv4 lease.

Resolution:

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

This triggered a fresh DHCP negotiation and restored the reserved address.

```text
Repository Structure
Project-Argus/
│
├── README.md
├── PROJECT.md
├── Architecture.md
├── CHANGELOG.md
├── LessonsLearned.md
├── Troubleshooting.md
│
├── ansible/
│   ├── inventory/
│   │   └── hosts.ini
│   ├── playbooks/
│   │   ├── install-common-tools.yml
│   │   └── install-docker.yml
│   └── roles/
│
├── docs/
│   ├── installation/
│   ├── networking/
│   ├── automation/
│   ├── operations/
│   └── virtualization/
│
├── monitoring/
│   ├── icinga/
│   └── grafana/
│
├── diagrams/
├── incidents/
└── screenshots/
```

---


## Roadmap

Next
 Deploy Project Guardian on docker01
 Validate ARM64 → x86-64 portability
 Validate Project Guardian using Docker Compose
 Document container deployment resource usage

Automation
 Refactor Ansible playbooks into roles
 Automate system baseline configuration
 Automate Docker application deployment
 Explore Proxmox API automation
 Automate VM provisioning

Monitoring
 Deploy Icinga
 Deploy Prometheus
 Deploy Node Exporter
 Deploy Grafana
 Monitor Proxmox host resources
 Monitor VM resources
 Monitor Docker services
 Integrate temperature and fan metrics

Security
 Automate TLS certificate deployment with Ansible
 Introduce Ansible Vault
 Deploy internal CA test environment
 Automate certificate rotation
 Explore ACME-based certificate renewal

Disaster Recovery
 Restore legacy virtual machine backups
 Rebuild Docker workloads from Git
 Test Project Guardian recovery
 Measure infrastructure recovery time

---

## Engineering Principles

Project Argus follows a few simple rules:

1.Understand before automating.
2.Implement small changes.
3.Validate every change.
4.Troubleshoot methodically.
5.Document failures as carefully as successes.
6.Prefer reproducible infrastructure.
7.Use automation to define desired state.
8.Measure infrastructure behavior whenever possible.

---

## Related Project

Project Argus is designed to complement Project Guardian, a home-network security and infrastructure project.
Future integration will use Project Argus as a virtualization, automation and monitoring platform for selected Guardian workloads.

---

## Status

Current development stage:

Infrastructure foundation + Ansible automation + Docker host provisioning

Next milestone:

Deploy and validate Project Guardian on `docker01`.
