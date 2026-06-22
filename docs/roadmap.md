# Roadmap

## Purpose

This document captures the planned direction for `basement-node`.

The goal is to keep future work organized around infrastructure learning, operational maturity, secure access, observability, and self-hosted cloud patterns.

This roadmap should help avoid drifting into unrelated troubleshooting or personal-use service work when the main goal is infrastructure growth.

## Current Baseline

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

## Priority 1: Documentation Quality

Current goal:

* Keep the documentation useful for future troubleshooting and technical review.
* Maintain notes that explain what exists, why it exists, how it was validated, and what to check if something breaks.

Planned improvements:

* Keep README current
* Add architecture notes
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

## Priority 3: Secure Browser-Based Access

Current remote access model:

* Tailscale private access
* UFW deny-incoming by default
* LAN access allowed from `private LAN subnet`

Planned next step:

* Add Cloudflare Tunnel and Cloudflare Access for selected browser-based services.

Preferred first protected service:

```text
grafana.pocketwhalegaming.com -> Grafana on basement-node
```

Reason:

* Grafana is infrastructure-aligned
* It is useful for review and troubleshooting
* It is safer than exposing a remote desktop/control path first
* It provides a clean pattern for future protected services

Future protected service examples:

```text
grafana.pocketwhalegaming.com -> protected Grafana access
remote.pocketwhalegaming.com  -> protected remote operations path
cloud.pocketwhalegaming.com   -> Nextcloud
status.pocketwhalegaming.com  -> service status page
```

Guiding rule:

Do not expose browser-based services directly to the public internet without an access-control layer.

## Priority 4: Self-Hosted Cloud Service

Planned service:

```text
Nextcloud
```

Purpose:

* Build experience with self-hosted cloud-style applications
* Practice persistent application deployment
* Practice storage planning
* Practice authentication and access control
* Practice backup/recovery planning for application data
* Practice reverse proxy / tunnel / protected access patterns

Nextcloud should come after the access model is stable enough to protect it properly.

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

* Finish cleaned documentation set
* Review README links
* Confirm monitoring stack still runs
* Confirm Grafana is reachable
* Confirm Tailscale status
* Confirm UFW posture
* Keep private storage unmounted during review
* Decide whether to initialize a Git repository
* Decide whether to publish to GitHub
* Plan Cloudflare Tunnel + Access implementation
* Plan Nextcloud deployment after secure access pattern is stable

## Summary

The roadmap for `basement-node` is to grow from a working Linux homelab into a more mature self-hosted infrastructure environment.

The focus is operational maturity: documentation, observability, secure access, backup/recovery, self-hosted cloud services, automation, and eventually cloud-native and AI infrastructure concepts.
