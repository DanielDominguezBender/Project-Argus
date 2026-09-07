# Proxmox Host Thermal Monitoring

## Test Environment

- MacBook Pro Late 2012
- Closed-lid operation
- Vertical stand
- Proxmox VE
- One or more lightweight Linux VMs
- Internal fan near minimum RPM

## Baseline — No External Cooling

| Metric | Value |
|---|---:|
| CPU Package | 60°C |
| Core 0 | 60°C |
| Core 1 | 57°C |
| Internal Fan | ~2000 RPM |
| Battery | 34.5°C |
| TPCD | 63°C |

## Mars Gaming MNBC2 Cooling

Measurements after more than 30 minutes:

| Metric | Value |
|---|---:|
| CPU Package | 58°C |
| Core 0 | 58°C |
| Core 1 | 56°C |
| Internal Fan | ~2000 RPM |
| Battery | 33.5°C |
| TPCD | 60°C |

## Preliminary Result

External cooling reduced several temperature readings by approximately 1–3°C while the internal fan remained at its minimum operating speed.

Further controlled measurements will be performed under CPU load.
