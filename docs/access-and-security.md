# Access and Security

## Purpose

This document describes the current access and security posture for `basement-node`.

The goal is to document how the system is accessed, what network paths are intentionally allowed, what is blocked by default, and how future browser-based access should be protected.

This document is intended to support future troubleshooting, review, and service expansion.

## Current Access Model

`basement-node` currently uses two trusted access paths:

```text
Local LAN access
Tailscale private access
```

The current model is intentionally simple:

* Allow trusted local network access from the home LAN
* Allow private remote access through Tailscale
* Deny unsolicited incoming traffic by default
* Avoid broad public exposure of services
* Add stronger identity-aware access controls before exposing browser-accessible services externally

## Network Baseline

Primary LAN:

```text
192.168.50.0/24
```

Known LAN IP:

```text
192.168.50.10
```

Known Tailscale IP:

```text
100.125.249.25
```

Tailscale is currently the private management plane for the system.

## Firewall Baseline

UFW is enabled.

Observed firewall state during the 2026-06-16 system review:

```text
Status: active
Logging: on (low)
Default: deny incoming, allow outgoing, deny routed
```

Allowed access:

```text
Anywhere on tailscale0      ALLOW IN    Anywhere
Anywhere                    ALLOW IN    192.168.50.0/24
Anywhere (v6) on tailscale0 ALLOW IN    Anywhere (v6)
```

This means the system currently allows:

* Traffic over the Tailscale interface
* Traffic from the trusted local LAN
* Outbound traffic from the host

It denies unsolicited incoming traffic by default outside those allowed paths.

## Tailscale Role

Tailscale provides private network access to `basement-node`.

Current Tailscale identity:

```text
100.125.249.25  basement-node
```

Operational role:

* Private remote management access
* Safer alternative to opening management ports publicly
* Useful path for reaching services while away from the local LAN
* Foundation for controlled administrative access

Tailscale should be treated as the current trusted remote access layer.

## Docker Service Exposure

The current monitoring stack publishes several ports on the host:

```text
Grafana             3000/tcp
Prometheus          9090/tcp
cAdvisor            8080/tcp
node-exporter       9100/tcp
blackbox-exporter   9115/tcp
```

These ports are intended for trusted LAN and private-access use.

They should not be treated as public internet services in their current form.

Before any browser-accessible service is exposed externally, it should be placed behind an identity-aware access layer such as Cloudflare Access.

## Current Security Posture

Current strengths:

* UFW is active
* Default incoming traffic is denied
* Tailscale is available for private access
* LAN access is explicitly scoped to `192.168.50.0/24`
* Services are not being intentionally exposed directly to the public internet
* Private bulk storage was unmounted during documentation/review preparation to reduce accidental data exposure

Current limitations:

* Some services publish ports on all host interfaces
* Grafana and Prometheus are currently suitable for trusted/private network access, not direct public exposure
* Service-level authentication and reverse proxy rules should be reviewed before broader access
* External browser access has not yet been placed behind Cloudflare Access

## Review Commands

Check Tailscale status:

```fish
tailscale status
```

Check UFW firewall status:

```fish
sudo ufw status verbose
```

Check running containers and published ports:

```fish
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"
```

Check the monitoring stack specifically:

```fish
cd /opt/stacks/monitoring
docker compose ps
```

Check current IP addresses:

```fish
ip addr
```

## Troubleshooting Flow

If a service is not reachable from the LAN:

1. Confirm the service container is running.
2. Confirm the expected port is published.
3. Confirm UFW allows LAN traffic from `192.168.50.0/24`.
4. Confirm the client is actually on the trusted LAN.
5. Confirm the service itself is listening correctly.

If a service is not reachable over Tailscale:

1. Confirm Tailscale is running on `basement-node`.
2. Confirm the client device is connected to the same Tailnet.
3. Confirm `tailscale status` shows expected peers.
4. Confirm UFW allows traffic on `tailscale0`.
5. Confirm the service port is published and listening.

If a service should not be reachable:

1. Confirm whether the port is published by Docker.
2. Confirm whether UFW allows access from the source network.
3. Confirm whether the service is bound to all interfaces.
4. Consider restricting the service to private network paths or placing it behind an access proxy.

## Cloudflare Tunnel and Access Roadmap

A future improvement is to add Cloudflare Tunnel and Cloudflare Access for identity-aware browser access.

The intended pattern is:

```text
User browser
    |
Cloudflare Access policy
    |
Cloudflare Tunnel
    |
Internal service on basement-node
```

The first recommended service to protect this way is Grafana because it is useful, infrastructure-aligned, and lower risk than exposing a remote desktop/control path.

Potential future service names:

```text
grafana.pocketwhalegaming.com -> protected Grafana access
remote.pocketwhalegaming.com  -> protected remote operations path
cloud.pocketwhalegaming.com   -> Nextcloud
status.pocketwhalegaming.com  -> service status page
```

This model avoids directly opening inbound public ports to the host.

## Access Design Principles

Guiding principles for this system:

* Prefer private access over public exposure
* Deny unsolicited inbound traffic by default
* Keep management paths controlled
* Use identity-aware access before exposing browser-based services externally
* Document what is exposed and why
* Avoid exposing private storage paths or unrelated personal-use services during review
* Treat access changes as infrastructure changes that should be validated and documented

## Future Improvements

Planned access/security improvements:

* Review which Docker services need host-published ports
* Consider binding some services only to localhost or private interfaces
* Add Cloudflare Tunnel for selected services
* Add Cloudflare Access authentication policies
* Start with protected Grafana access
* Add protected access for Nextcloud after deployment
* Document service-specific access requirements
* Add a simple access runbook
* Review whether additional UFW rules should replace broad LAN allowance for specific services

## Summary

`basement-node` currently uses a simple private-first access model: trusted LAN access, Tailscale private remote access, and UFW deny-incoming by default.

The next major security improvement is to add Cloudflare Tunnel and Cloudflare Access for identity-aware browser access to selected internal services without directly exposing the host to the public internet.
