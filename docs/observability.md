# Observability

## Purpose

This document describes the observability stack running on `basement-node`.

The purpose of the stack is to provide visibility into the health and behavior of the host, containers, and key services. It is intended to support day-to-day review, troubleshooting, capacity awareness, and future service expansion.

This stack also serves as a practical learning environment for infrastructure monitoring patterns using Docker, Prometheus, Grafana, and exporters.

## Documentation Status

The container-health and host-published port details below were observed
during the 2026-06-16 system review. Later project history records a reduction
in monitoring exposure: unnecessary host-published ports were removed for
Prometheus, node-exporter, cAdvisor, and blackbox-exporter, while Grafana
remained intentionally reachable as the primary monitoring UI. No new
live-system inspection was performed for this update.

## Stack Location

The monitoring stack is located at:

```text
/opt/stacks/monitoring
```

It is managed with Docker Compose.

Primary validation command:

```fish
cd /opt/stacks/monitoring
docker compose ps
```

Secondary validation command:

```fish
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"
```

## Documented Services

The documented stack includes:

```text
prometheus
grafana
node-exporter
cadvisor
blackbox-exporter
```

At the time of the 2026-06-16 system review, all five monitoring containers were running and had been up for approximately six days.

## Service Roles

### Grafana

Grafana is the dashboard and visualization layer.

It provides a browser-based view into host and service metrics. This is the main place to review system behavior visually.

Service port recorded during the 2026-06-16 review:

```text
3000/tcp
```

Common uses:

* Review host CPU, memory, disk, and network activity
* Review container-level behavior if dashboards are configured
* Quickly check for unusual resource usage
* Provide a visual summary of system health

### Prometheus

Prometheus is the metrics collection and query layer.

It scrapes metrics from configured exporters and stores time-series data that Grafana can visualize.

Service port recorded during the 2026-06-16 review:

```text
9090/tcp
```

Common uses:

* Check scrape target health
* Query raw metrics
* Confirm exporters are reachable
* Troubleshoot missing dashboard data

### node-exporter

node-exporter exposes Linux host metrics to Prometheus.

Service port recorded during the 2026-06-16 review:

```text
9100/tcp
```

Useful host-level metrics include:

* CPU usage
* memory usage
* disk usage
* filesystem capacity
* network activity
* system load
* uptime

If host-level metrics are missing from Grafana, node-exporter and Prometheus target health should be checked first.

### cAdvisor

cAdvisor exposes container-level metrics.

Service port recorded during the 2026-06-16 review:

```text
8080/tcp
```

Useful container-level metrics include:

* container CPU usage
* container memory usage
* container uptime
* container resource pressure
* container lifecycle behavior

If container metrics are missing or stale, check whether the `cadvisor` container is running and healthy.

### blackbox-exporter

blackbox-exporter supports reachability checks.

Service port recorded during the 2026-06-16 review:

```text
9115/tcp
```

Useful checks include:

* Whether a service endpoint responds
* Whether an internal URL is reachable
* Whether a target is down from the monitoring stack’s point of view

This will become more useful as additional self-hosted services are added.

## Current Validation State

Observed Docker Compose state during the 2026-06-16 system review:

```text
blackbox-exporter   Up 6 days
cadvisor            Up 6 days (healthy)
grafana             Up 6 days
node-exporter       Up 6 days
prometheus          Up 6 days
```

Observed Docker container state:

```text
prometheus          Up 6 days
blackbox-exporter   Up 6 days
grafana             Up 6 days
cadvisor            Up 6 days (healthy)
node-exporter       Up 6 days
```

This confirms that the monitoring stack was persistent and stable across multiple days of normal system use.

## Historical Published Ports

**HISTORICAL STATE — 2026-06-16:** these service ports were published on the
host:

```text
Grafana             3000/tcp
Prometheus          9090/tcp
cAdvisor            8080/tcp
node-exporter       9100/tcp
blackbox-exporter   9115/tcp
```

These ports were intended for trusted LAN and private-access use, not broad
public exposure.

This list is preserved as dated evidence and is not the current exposure
model.

## Later Exposure Reduction

Later project history records the following state:

* The unnecessary Prometheus host-published port was removed.
* The unnecessary node-exporter host-published port was removed.
* The unnecessary cAdvisor host-published port was removed.
* The unnecessary blackbox-exporter host-published port was removed.
* Grafana remains intentionally reachable as the primary monitoring UI.
* Internal monitoring health checks passed after the exposure reduction.

The tracked evidence does not establish exact current Compose mappings,
bindings, or Grafana's access route. Those details require live validation and
are intentionally not inferred here.

## Docker and UFW Boundary

Docker-published ports require review as their own exposure boundary. Docker
forwarding and NAT behavior can allow published container ports to traverse
paths that do not match expected UFW host filtering. This should not be
shortened to "UFW does not work with Docker": UFW still governs host policy,
while Docker publication, interface binding, forwarding policy, and upstream
network controls together determine container reachability.

The preferred monitoring design is to keep scrape-only components internal to
the monitoring network and expose only an intentional user-facing entry point.

## Review / Troubleshooting Flow

For a quick review or troubleshooting session, a useful sequence is:

1. Check Docker Compose service state.
2. Confirm the expected monitoring containers are running.
3. Open Grafana.
4. Review host-level metrics.
5. Review container-level metrics if available.
6. Check Prometheus target health if a scrape issue is suspected.
7. Use exporter roles to narrow whether the issue is host-level, container-level, or endpoint-level.
8. Record any findings in the project notes.

## Basic Commands

Check the monitoring stack:

```fish
cd /opt/stacks/monitoring
docker compose ps
```

Check running containers and exposed ports:

```fish
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"
```

Check host resource state:

```fish
free -h
df -h /
```

Check whether Docker is available:

```fish
docker --version
docker compose version
```

## What to Check if Grafana Is Not Reachable

Check whether the Grafana container is running:

```fish
cd /opt/stacks/monitoring
docker compose ps grafana
```

Check all monitoring containers:

```fish
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"
```

Inspect Grafana's current bindings and published-port state:

```fish
docker ps --format "table {{.Names}}\t{{.Ports}}" | grep grafana
```

Compare the result with the intended Grafana route. The repository records
Grafana as intentionally reachable, but it does not prove whether the current
path is LAN, Tailscale, localhost plus a tunnel, or another controlled route.

Check whether the host firewall posture changed:

```fish
sudo ufw status verbose
```

If accessing remotely, confirm Tailscale is connected:

```fish
tailscale status
```

## What to Check if Dashboards Are Empty

If Grafana opens but dashboards are empty or missing data:

1. Confirm Prometheus is running.
2. Check Prometheus through an approved administrative or container-internal path.
3. Check Prometheus target health.
4. Confirm `node-exporter` is running for host metrics.
5. Confirm `cadvisor` is running and healthy for container metrics.
6. Check whether dashboard queries match the available Prometheus labels.
7. Check recent container restarts or configuration changes.

Useful command:

```fish
cd /opt/stacks/monitoring
docker compose ps
```

## Operational Notes

* Grafana is the visual layer.
* Prometheus is the metrics/query layer.
* node-exporter represents the Linux host.
* cAdvisor represents Docker/container behavior.
* blackbox-exporter represents endpoint reachability.
* Missing data in Grafana does not always mean the service is down; it may mean Prometheus is not scraping the expected target or the dashboard query does not match current labels.
* Container uptime is useful for confirming whether services have been stable or recently restarted.

## Future Improvements

Planned observability improvements:

* Add clearer Grafana dashboard organization
* Add dashboard descriptions and notes
* Add alerting for disk, memory, and service health
* Add blackbox checks for key internal endpoints
* Add a simple uptime/status page
* Add monitoring documentation for each service
* Add a runbook for restarting or troubleshooting the stack
* Confirm and document the intended Grafana access route and its controls
* Add monitoring coverage for deployed services such as Nextcloud

## Summary

The observability stack is one of the core operational components of `basement-node`.

It provides visibility into host health, container behavior, service state, and future endpoint availability. It also creates a practical environment for learning monitoring workflows used in infrastructure, platform operations, cloud operations, and reliability-focused engineering.

The 2026-06-16 host-published port list is retained as historical evidence.
Later documentation records reduced exposure for the internal monitoring
components while preserving intentional access to Grafana.
