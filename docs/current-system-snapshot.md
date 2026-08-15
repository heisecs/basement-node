# Current System Snapshot

## Purpose

This document records the current operational baseline for `basement-node`.

It is intended as a concise reference for troubleshooting, maintenance planning, hardware changes, and documentation catch-up work. It should describe the currently verified state of the system, not serve as an incident log, package-maintenance history, or roadmap.

Last verified baseline: August 2026 documentation catch-up.

## Summary

`basement-node` is a Linux server/workstation running Pop!_OS 24.04 LTS with KDE Plasma on X11.

Current baseline:

- CPU: Intel i7-6950X, 10 cores / 20 threads
- Motherboard: MACHINIST X99 PR9-H
- RAM: 64 GB DDR4 using 4x16 GB SK hynix ECC-capable DIMMs
- ECC status: ECC-capable memory installed, but ECC is not active on the current motherboard/platform
- GPU: XFX Radeon RX 6600 XT 8 GB
- Kernel: `6.18.7-76061807-generic`
- Root storage: approximately 907 GB NVMe
- Bulk storage: separate ext4-mounted media/application storage
- Remote access: LAN SSH, Tailscale SSH, SFTP, Moonlight/Sunshine, and Cloudflare Access/Tunnel with noVNC/x11vnc
- Monitoring: Docker-based Prometheus/Grafana stack
- Rollback: Timeshift local snapshots

Planned PSU, GPU, and motherboard upgrades have not yet been installed.

## System Baseline

Host:

```text
basement-node
```

Operating system:

```text
Pop!_OS 24.04 LTS
```

Desktop/session baseline:

```text
KDE Plasma on X11
```

Shell preference:

```text
fish
```

Kernel:

```text
6.18.7-76061807-generic
```

## Hardware Baseline

### CPU

```text
Intel i7-6950X
10 cores / 20 threads
```

### Motherboard

```text
MACHINIST X99 PR9-H
```

Current platform notes:

- LGA2011-3 / X99-era platform
- Limited PCIe expansion flexibility compared with desired future platform
- ECC functionality is not active on this board/platform
- Current board remains installed

### Memory

Current installed memory:

```text
64 GB DDR4
4x16 GB SK hynix
2Rx8
PC4-2400T-EE1-11
Part number: HMA82GU7AFR8N-UH
```

Operating state:

```text
RAM configured around 2133 MT/s for stability
ECC-capable DIMMs installed
ECC not active on current motherboard/platform
```

Validation completed:

- BIOS/OS detection confirmed
- `stress-ng` 10-minute test using approximately 32 GB passed
- `stress-ng` 30-minute test using approximately 40 GB passed
- `memtester` 40 GB, one full pass, passed
- No MCE/EDAC errors observed during validation

## Graphics Baseline

Current installed GPU:

```text
XFX Radeon RX 6600 XT 8 GB
```

Current graphics baseline:

```text
Kernel driver: amdgpu
OpenGL renderer: AMD Radeon RX 6600 XT
Mesa: 25.2.8-0ubuntu0.24.04.1
PCIe link: 16 GT/s x16
```

This is the known-good pre-upgrade graphics baseline.

The planned R9700 GPU has not been installed.

## Storage Baseline

Root storage:

```text
~907 GB NVMe
```

Bulk/application storage:

```text
Separate ext4-mounted storage volumes
```

Current storage use includes:

- OS and applications on NVMe root storage
- Larger application/media/service data on separate ext4 storage
- Nextcloud data currently placed on secondary bulk storage
- Monitoring stack data managed under the Docker monitoring stack

Operational boundary:

```text
Timeshift provides local system rollback.
Timeshift is not an independent data backup solution.
```

## Remote Access Baseline

Current administrative access methods:

```text
LAN SSH
Tailscale SSH
SFTP through SSH
Moonlight/Sunshine
Cloudflare Access/Tunnel with noVNC/x11vnc
```

Cloudflare/noVNC access path:

```text
Cloudflare Access/Tunnel
→ noVNC/websockify
→ x11vnc
→ KDE/X11 desktop
```

Current verified state:

- Browser-based desktop access through Cloudflare Access/Tunnel works
- Phone SSH over Tailscale has been validated as an emergency administrative path
- LAN SSH remains available and preferred while on the home network
- SFTP through SSH has been tested for phone-to-server file transfer
- `cloudflared` runs as a systemd service

Known implementation debt:

- noVNC currently uses a self-referential symlink workaround
- Cloudflare tunnel token hygiene should be remediated without exposing token contents

## Docker and Service Baseline

Docker is installed and used for service stacks.

Current notable stacks/services:

```text
Monitoring stack
Nextcloud stack
```

Monitoring components:

```text
Prometheus
Grafana
node-exporter
cAdvisor
blackbox-exporter
```

Current monitoring exposure model:

- Grafana remains intentionally reachable as the primary monitoring UI
- Unnecessary host-published ports were removed for internal monitoring components
- Internal monitoring health checks passed after reducing unnecessary host-published monitoring ports

Nextcloud current state:

- Nextcloud is deployed as a Docker Compose stack
- Public access is routed through Cloudflare Tunnel
- Local origin is bound to localhost
- Phone auto-upload has been validated
- This is not yet a complete backup or disaster-recovery architecture

## Firewall and Security Posture

Current security posture:

- UFW is active
- Default inbound policy is deny
- Trusted local and remote administration paths are intentionally allowed
- Public service exposure is handled through Cloudflare Tunnel where applicable
- Tailscale remains available for private remote administration
- Docker-published port exposure has been reviewed and reduced for the monitoring stack

Important boundary:

```text
Docker-published ports can bypass expected UFW host filtering through Docker forwarding behavior.
```

Operational security rules for public documentation:

- Do not publish Cloudflare tunnel tokens
- Do not publish private keys
- Do not publish passwords or database credentials
- Do not publish raw production configuration files containing secrets
- Do not publish unnecessary private topology or device inventory

## Package Maintenance Baseline

Current package-maintenance approach:

```text
Use controlled package-family updates
Simulate upgrades before applying
Avoid blanket upgrades
Validate services after each package family
Defer high-risk platform changes to deliberate maintenance windows
```

Explicitly avoided as routine maintenance:

```text
apt full-upgrade
blanket apt upgrade
apt autoremove
automatic kernel upgrades
automatic graphics-stack upgrades
```

## Deferred High-Risk Update Families

The following update families remain intentionally deferred pending deliberate maintenance windows:

```text
kernel
linux-firmware
Mesa / graphics stack
libdrm / libva / VA drivers
GStreamer / media stack
Xorg / Xwayland
COSMIC / Mutter / desktop portal packages
systemd / systemd-boot / udev
NetworkManager
System76 graphics/power/DKMS-related packages
glibc/libc
Samba
```

Reasons include:

- Preserve the known-good RX 6600 XT graphics baseline
- Avoid disrupting remote access and graphical-session troubleshooting
- Avoid combining graphics, firmware, kernel, and hardware changes
- Preserve rollback safety before high-risk system changes

## Backup and Rollback Baseline

Timeshift is used for local rollback snapshots.

Current policy:

- Create a fresh Timeshift snapshot before high-risk package families
- Create a fresh Timeshift snapshot before major platform changes
- Preserve a rollback path before PSU, GPU, graphics, kernel, or system-level changes
- Validate alternate access paths where relevant

Boundary:

```text
Timeshift is local rollback.
Timeshift is not a substitute for independent data backup.
```

Nextcloud backup/restore automation remains future work.

## Known Open Issues

Current known open items:

- Full root cause of the earlier KDE/X11 remote graphical-session incident is not conclusively proven
- Sunshine/KWin/X11 degraded capture behavior remains under investigation
- noVNC symlink workaround should be cleaned up
- Cloudflare tunnel token hygiene should be remediated without exposing token contents
- Display resolution persistence after reboot remains unresolved
- Bluetooth instability remains under investigation; replacement adapter is planned
- OpenSSH `/run/sshd` recovery incident still needs a dedicated factual writeup
- Monitoring documentation may need reconciliation with the current Docker/UFW exposure model
- Nextcloud still needs tested backup and restore procedures before it should be treated as a real backup system

## Planned Changes Not Yet Installed

### PSU

Planned PSU:

```text
EVGA SuperNOVA 850 P6 Platinum
Model: 220-P6-0850-X1
```

Current status:

```text
Purchased, not installed
```

### GPU

Purchased GPU:

```text
Gigabyte AMD Radeon AI PRO R9700 AI TOP 32G
32 GB VRAM
```

Current status:

```text
Purchased, not installed
RX 6600 XT remains the active installed GPU
```

### Motherboard

Preferred future motherboard:

```text
Supermicro X10SRA-F
```

Current status:

```text
Planned future migration only
MACHINIST X99 PR9-H remains installed
```
