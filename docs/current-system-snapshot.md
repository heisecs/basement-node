# Current System Snapshot

## Purpose

This document captures the current baseline state of `basement-node` as of 2026-06-16.

The goal is to document the system in a way that is useful for infrastructure review: hardware, operating system, storage, Docker services, access model, firewall posture, and recovery state.

## System Identity

* Hostname: `basement-node`
* Primary user: `sona`
* Operating system: Pop!_OS 24.04 LTS
* Kernel: Linux 6.18.7-76061807-generic
* Architecture: x86-64
* Chassis: desktop
* Hardware vendor: MACHINIST
* Hardware model: X99 PR9-H
* Firmware version: 5.11

## Hardware Baseline

* Platform: MACHINIST X99 PR9-H
* GPU: AMD Radeon RX 6600 XT
* Memory installed: 32 GB
* Linux-visible memory: approximately 31 GiB
* RAM speed: 2133 MT/s
* Root drive: NVMe
* Root filesystem: ext4

Memory was intentionally left at the default 2133 MT/s instead of enabling XMP. The priority for this system is stability and predictable operation.

## Operating System Resource Snapshot

Observed memory state:

```text
Mem: 31Gi total, 13Gi used, 1.4Gi free, 19Gi buff/cache, 17Gi available
Swap: 19Gi total, 3.8Gi used, 16Gi free
```

Observed root filesystem state:

```text
Filesystem: /dev/nvme0n1p3
Size: 907G
Used: 160G
Available: 702G
Use: 19%
Mounted on: /
```

## Storage Layout

Current storage devices:

```text
nvme0n1p3  ext4  root filesystem  mounted at /
sda2       ext4  media-primary     not mounted during this review session
sdb2       ext4  media-secondary   mounted at /mnt/media-secondary
```

Current privacy posture:

* The private bulk-storage volume labeled `media-primary` was cleanly unmounted before documentation
* The secondary volume remains mounted at `/mnt/media-secondary`.
* The environment is intended to focus on infrastructure services, monitoring, access control, and documentation rather than private datasets.

## Docker Baseline

Docker is installed and active.

Observed versions:

```text
Docker version 29.5.1
Docker Compose version v5.1.3
```

## Monitoring Stack

The monitoring stack is located at:

```text
/opt/stacks/monitoring
```

Current Dockerized monitoring services:

```text
prometheus
grafana
blackbox-exporter
cadvisor
node-exporter
```

Observed state during validation:

```text
prometheus          Up 6 days
grafana             Up 6 days
blackbox-exporter   Up 6 days
cadvisor            Up 6 days (healthy)
node-exporter       Up 6 days
```

Published ports:

```text
Grafana             3000/tcp
Prometheus          9090/tcp
cAdvisor            8080/tcp
node-exporter       9100/tcp
blackbox-exporter   9115/tcp
```

## Network and Access

Primary LAN:

```text
192.168.50.0/24
```

Known LAN IP:

```text
192.168.50.10
```

Tailscale is installed and working.

Known Tailscale IP:

```text
100.125.249.25
```

Tailscale is used as the private management plane for remote access.

## Firewall Posture

UFW is enabled.

Observed UFW posture:

```text
Status: active
Logging: on (low)
Default: deny incoming, allow outgoing, deny routed
```

Allowed access:

```text
Allow traffic on tailscale0
Allow traffic from 192.168.50.0/24
```

This provides a simple intentional access model:

* Trusted LAN access from the local network
* Private remote management through Tailscale
* Default deny for unsolicited incoming traffic outside allowed paths

## Backup and Recovery State

Timeshift is configured in RSYNC mode.

Observed state:

```text
Status: OK
10 snapshots
753.3 GB free
```

Important retained snapshots:

```text
2026-05-19_21-13-42  Known good after Docker Tailscale UFW setup with trimmed excludes
2026-06-09_16-56-53  Known good after RAM upgrade to 32GB
2026-06-16_14-45-51  pre-demo-basement-node-portfolio-prep
```

The pre-documentation snapshot was created before preparation. This supports a rollback-aware operating model.

## Current Review-Safe State

For documentation, troubleshooting, or technical review, the intended focus areas are:

* Documentation and system architecture
* Docker monitoring stack
* Grafana dashboard
* Prometheus-backed observability
* Tailscale private access
* UFW firewall posture
* Timeshift backup/recovery policy
* Roadmap toward Cloudflare Access and Nextcloud

This intentionally avoids exposing private storage paths or unrelated personal-use services.

## Summary

`basement-node` is currently a stable self-hosted Linux infrastructure lab with Dockerized observability, private management access, firewall controls, documented storage posture, and Timeshift-based recovery points.

It provides a practical environment for building skills in host administration, service ownership, monitoring, secure access, change validation, and infrastructure documentation.
