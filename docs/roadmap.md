# Roadmap

## Purpose

This document captures the planned direction for `basement-node`.

The goal is to keep future work organized around infrastructure learning, operational maturity, secure access, observability, and self-hosted cloud patterns.

This roadmap should help avoid drifting into unrelated troubleshooting or personal-use service work when the main goal is infrastructure growth.

This is a repository planning record, not proof of current live-system state. Completed milestones below reflect documented project history; live status requires separate validation.

## Documented Baseline

`basement-node` currently provides:

* Linux host administration practice
* Dockerized monitoring stack
* Prometheus metrics collection
* Grafana dashboarding
* Host metrics through node-exporter
* Container metrics through cAdvisor
* Endpoint reachability foundation through blackbox-exporter
* Private remote access through Tailscale
* Firewall control through UFW
* Backup and rollback points through Timeshift
* Hardware upgrade validation documentation
* Cleaned infrastructure documentation

## Completed Milestones

The following work is complete in the documented project history and is no longer future backlog:

* Git repository initialization and ongoing maintenance
* Publication of the public GitHub repository
* Cloudflare Tunnel deployment
* Cloudflare Access deployment for selected browser-facing services
* noVNC/websockify/x11vnc browser desktop deployment
* Nextcloud deployment
* EVGA SuperNOVA 850 P6 and R9700 installation
* ROCm 7.2.4 container, HIP, and PyTorch ROCm validation
* llama.cpp HIP/`gfx1201` build validation
* Creation of the permanent `/opt/stacks/llama-rocm` stack

The June 2026 RAM upgrade from 16 GB to 32 GB remains a valid dated historical milestone. It was later superseded by the documented 64 GB baseline. See [Hardware Upgrade and Validation Workflow](hardware-upgrade-validation.md) and [Current System Snapshot](current-system-snapshot.md).

Detailed PSU/GPU and local-AI validation belongs in [Power and R9700 Upgrade Validation](power-and-r9700-upgrade-validation.md) and [ROCm and llama.cpp Stack](rocm-and-llama-cpp-stack.md), rather than being duplicated here.

## Priority 1: Documentation Quality

Current goal:

* Keep the documentation useful for future troubleshooting and technical review.
* Maintain notes that explain what exists, why it exists, how it was validated, and what to check if something breaks.

Planned improvements:

* Keep README current
* Maintain the architecture overview as the system evolves
* Add service-specific runbooks
* Add change logs for meaningful infrastructure updates
* Keep private/personal-use details out of the main infrastructure docs
* Add diagrams where they make the system easier to understand

## Priority 2: Observability Maturity

Current monitoring stack:

* Prometheus
* Grafana
* node-exporter
* cAdvisor
* blackbox-exporter

Planned improvements:

* Organize Grafana dashboards more clearly
* Add dashboard descriptions
* Add blackbox checks for key internal services
* Add alerting for disk pressure, memory pressure, and service failures
* Add a simple uptime/status view
* Document what each dashboard is expected to show
* Create a monitoring troubleshooting runbook

## Priority 3: Secure Browser-Based Access Maintenance

Current remote access model:

* Tailscale private access
* UFW deny-incoming by default
* LAN SSH available for local administration
* Cloudflare Tunnel for selected browser-facing services
* Cloudflare Access for selected identity-gated browser access
* noVNC/websockify/x11vnc browser desktop path

Remaining work:

* Revalidate current Tunnel and Access policy enforcement without exposing connector authentication material
* Remediate Cloudflare tunnel token hygiene
* Clean up the noVNC symlink/workaround if it remains present
* Continue observing x11vnc stability after the `-noxfixes` change
* Preserve Bluetooth and Fire Stick/Moonlight findings as observations until repeated validation supports stronger conclusions
* Write a Docker/UFW hardening runbook after the live exposure model is validated

Guiding rule:

Do not expose browser-based services directly to the public internet without an access-control layer.

## Priority 4: Self-Hosted Cloud Service Recovery

Nextcloud is deployed as a Docker Compose workload with a localhost-bound origin and Cloudflare Tunnel access. Phone auto-upload and server-side retention after phone-side deletion have been validated.

Remaining work:

* Create a durable procedure covering Nextcloud configuration, database, and data backup
* Automate backup and restore steps where appropriate
* Test restoration rather than treating successful backup creation as sufficient

Timeshift provides local system rollback. It is not an independent backup for Nextcloud application data or protection from storage-device failure.

## Priority 5: Operational Runbooks

Planned runbooks:

* Monitoring stack restart/check procedure
* Grafana unreachable troubleshooting
* Prometheus target health troubleshooting
* Tailscale access troubleshooting
* UFW rule review procedure
* Timeshift snapshot creation procedure
* Pre-change checklist
* Post-change validation checklist
* Storage mount/unmount checklist
* Service deployment checklist

These runbooks should be practical and short. The goal is to make future troubleshooting faster and safer.

## Priority 6: Infrastructure Automation

Potential future work:

* Convert more setup steps into repeatable scripts
* Use environment files for service configuration
* Standardize Docker Compose layouts
* Add backup scripts where appropriate
* Add health-check scripts
* Explore Ansible or similar configuration management tooling later

Local-AI automation should remain narrow until the operating pattern stabilizes. Supported future work includes optional endpoint health checks, reconciling the older `lich-model` CLI with router mode, and documenting a model-management workflow if one becomes stable.

Automation should come after the system is better documented and the desired service patterns are clearer.

## Priority 7: Platform / Cloud-Native Growth

Longer-term learning direction:

* Better Docker Compose structure
* Reverse proxy patterns
* Identity-aware access
* Persistent storage planning
* Monitoring and alerting
* Service health checks
* Infrastructure-as-code concepts
* Kubernetes fundamentals
* OCI/cloud infrastructure concepts
* GPU/AI infrastructure concepts over time
* Preferred future motherboard migration to the Supermicro X10SRA-F

This direction aligns `basement-node` with infrastructure engineering, platform operations, cloud operations, and eventually AI/GPU infrastructure reliability.

## What Not to Prioritize in Main Infrastructure Docs

The main documentation set should avoid centering:

* Personal media workflows
* VR playback workflows
* Gaming novelty
* Private datasets
* One-off desktop tinkering that does not support the infrastructure story

Those topics may exist elsewhere as raw notes, but the main infrastructure docs should remain focused on durable system operation and professional growth.

## Near-Term Checklist

Immediate next steps:

* Define and automate independent backup procedures where appropriate
* Document durable Nextcloud configuration, database, and data backup
* Perform and record restore testing
* Define backup expectations for local-AI models and caches
* Remediate Cloudflare tunnel token hygiene without exposing token contents
* Clean up the noVNC symlink/workaround if it remains present
* Continue x11vnc `-noxfixes`, Bluetooth, and Fire Stick/Moonlight observation
* Reconcile the older `lich-model` CLI with the current router-mode workflow
* Add optional monitoring or health checks for the localhost-only local-AI endpoint
* Create a Docker/UFW hardening runbook after live validation
* Document a model-management workflow if it stabilizes
* Plan the preferred future motherboard migration to the Supermicro X10SRA-F

## Summary

The roadmap for `basement-node` is to grow from a working Linux homelab into a more mature self-hosted infrastructure environment.

The focus is operational maturity: documentation, observability, secure access, backup/recovery, self-hosted cloud services, automation, and eventually cloud-native and AI infrastructure concepts.
