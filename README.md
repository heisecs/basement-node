# basement-node: Self-Hosted Linux Infrastructure Lab

## Purpose

`basement-node` is a self-hosted Linux infrastructure lab used to practice host administration, Docker service ownership, observability, secure access design, firewall posture, backup and recovery discipline, and infrastructure documentation.

The system is both a useful home server and a practical learning environment. The main documentation set focuses on durable infrastructure work: how the system is built, how it is monitored, how it is accessed, how it is recovered, and where it is going next.

## Project Goals

The current goals of this project are to build experience with:

* Linux infrastructure administration
* Dockerized service deployment
* Prometheus and Grafana observability
* Host and container monitoring
* Private remote access with Tailscale
* Firewall policy with UFW
* Backup and rollback planning with Timeshift
* Hardware upgrade validation
* Containerized GPU compute and local-AI service operation
* Clean technical documentation
* Secure browser-based access patterns
* Future self-hosted cloud services

Longer term, this project supports growth toward infrastructure engineering, deployment and provisioning work, host configuration, platform operations, cloud infrastructure, and AI/GPU infrastructure concepts.

## Current System Snapshot

Current high-level baseline:

* Hostname: `basement-node`
* Operating system: Pop!_OS 24.04 LTS
* Kernel: Linux 6.18.7-76061807-generic
* Hardware platform: MACHINIST X99 PR9-H
* GPU: Gigabyte AMD Radeon AI PRO R9700 AI TOP 32G
* GPU memory: 32 GB GDDR6
* Memory: 64 GB DDR4
* Root storage: NVMe
* Docker version: 29.5.1
* Docker Compose version: v5.1.3

For the full baseline, see:

* [Current System Snapshot](docs/current-system-snapshot.md)

## Local AI Stack

The active R9700 is used through a container-first ROCm architecture. The host provides `amdgpu` and the GPU device interfaces, while ROCm development tools, AI frameworks, and llama.cpp remain containerized.

The permanent llama.cpp stack is located at `/opt/stacks/llama-rocm`. Its web endpoint is bound to `127.0.0.1:8088` and is not publicly exposed.

For details, see:

* [Power and R9700 Upgrade Validation](docs/power-and-r9700-upgrade-validation.md)
* [ROCm and llama.cpp Stack](docs/rocm-and-llama-cpp-stack.md)

## Core Services

The current infrastructure-focused service stack is the Dockerized monitoring stack located at:

```text
/opt/stacks/monitoring
```

Current monitoring services:

* Prometheus
* Grafana
* node-exporter
* cAdvisor
* blackbox-exporter

At the time of the 2026-06-16 system review, all five monitoring containers were running and had been up for approximately six days.

For details, see:

* [Observability](docs/observability.md)

## Access and Security

The current access model uses:

* Trusted LAN access from `private LAN subnet`
* Private remote access through Tailscale
* UFW firewall policy
* Default deny for incoming traffic outside allowed paths

Current UFW posture:

```text
Default: deny incoming, allow outgoing, deny routed
```

Tailscale is treated as the current private management plane. Future browser-based service access should use an identity-aware access layer before anything is exposed externally.

For details, see:

* [Access and Security](docs/access-and-security.md)

## Backup and Recovery

Timeshift is configured and used for system snapshots.

Important retained recovery points include:

```text
2026-05-19_21-13-42  Known good after Docker Tailscale UFW setup with trimmed excludes
2026-06-09_16-56-53  Known good after RAM upgrade to 32GB
2026-06-16_14-45-51  pre-demo-basement-node-portfolio-prep
2026-08-17_00-12-34  Known-good post-R9700 + EVGA 850 P6 install, pre-ROCm
```

The recovery model is based on understanding the current state, making controlled changes, validating results, retaining rollback points, and documenting meaningful changes.

For details, see:

* [Backup and Recovery](docs/backup-and-recovery.md)

## Hardware Upgrade Validation

The June 2026 RAM upgrade from 16 GB to 32 GB remains documented as a historical hardware-validation workflow. The system was subsequently expanded to 64 GB, as recorded in the current snapshot.

Hardware history and current upgrade records:

* [Hardware Upgrade and Validation Workflow](docs/hardware-upgrade-validation.md)
* [Power and R9700 Upgrade Validation](docs/power-and-r9700-upgrade-validation.md)

## Data Exposure and Privacy Posture

The system contains storage and services that are useful locally but are not relevant to infrastructure review or troubleshooting.

During the 2026-06-16 documentation and review preparation session, the private bulk-storage volume was cleanly unmounted to prevent accidental exposure of data that is not relevant to the infrastructure work being documented.

Current storage posture from that session:

```text
sda2  ext4  media-primary    not mounted
sdb2  ext4  media-secondary  mounted at /mnt/media-secondary
```

The main documentation set intentionally focuses on the infrastructure layer:

* Host administration
* Docker service state
* Observability
* Access control
* Firewall posture
* Backup and recovery
* Roadmap and future service design

## Roadmap

Near-term direction:

* Keep documentation current and useful for troubleshooting
* Improve Grafana dashboard organization
* Add service-specific runbooks
* Add Cloudflare Tunnel and Cloudflare Access for selected browser-based services
* Start with protected Grafana access
* Add Nextcloud as the next major self-hosted cloud service
* Expand monitoring and alerting
* Build toward more repeatable service deployment patterns

For details, see:

* [Roadmap](docs/roadmap.md)

## Documentation Index

Core documentation:

* [Current System Snapshot](docs/current-system-snapshot.md)
* [Architecture Overview](docs/architecture-overview.md)
* [Observability](docs/observability.md)
* [Access and Security](docs/access-and-security.md)
* [Backup and Recovery](docs/backup-and-recovery.md)
* [Hardware Upgrade and Validation Workflow](docs/hardware-upgrade-validation.md)
* [Power and R9700 Upgrade Validation](docs/power-and-r9700-upgrade-validation.md)
* [ROCm and llama.cpp Stack](docs/rocm-and-llama-cpp-stack.md)
* [Roadmap](docs/roadmap.md)
* [Secure Remote Access and Custom Domain Email](docs/secure-remote-access-and-domain-email.md)

## Operating Principles

Guiding principles for this project:

* Keep changes controlled and documented
* Validate before assuming success
* Prefer private access over public exposure
* Use monitoring to understand system behavior
* Maintain rollback points before meaningful changes
* Keep infrastructure documentation separate from unrelated personal-use workflows
* Build toward repeatable, secure, observable service patterns

## Summary

`basement-node` is a working self-hosted Linux infrastructure and local-AI lab.

It demonstrates Linux administration, Docker service ownership, observability, private access, firewall policy, backup/recovery discipline, hardware validation, containerized ROCm operation, and GPU-accelerated llama.cpp workloads.
