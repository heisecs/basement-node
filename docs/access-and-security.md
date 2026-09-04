# Access and Security

## Purpose

This document preserves the 2026-06-16 access and security baseline for
`basement-node` and records later documented changes to that model.

The goal is to document how the system was accessed at each recorded point,
what network paths were intentionally allowed, what was blocked by default,
and how browser-based access was subsequently protected.

This document is intended to support future troubleshooting, review, and service expansion.

## Documentation Status

Status: **HISTORICAL STATE**

The firewall and network baseline in this document was observed during the
2026-06-16 system review. It should not be treated as verified live state
without a new system check.

The separate 2026-06-16 secure remote-access record documents Cloudflare
Tunnel and Cloudflare Access as configured. That dated result supersedes the
planning language below for that observation period, but does not prove that
the same configuration remains active.

## Historical Access Model

During the 2026-06-16 system review, `basement-node` used two trusted access
paths:

```text
Local LAN access
Tailscale private access
```

That model was intentionally simple:

* Allow trusted local network access from the home LAN
* Allow private remote access through Tailscale
* Deny unsolicited incoming traffic by default
* Avoid broad public exposure of services
* Add stronger identity-aware access controls before exposing browser-accessible services externally

This was later superseded by a broader documented model: Tailscale SSH and
SFTP over SSH were validated for remote administration and file transfer, and
Cloudflare Tunnel plus Cloudflare Access were deployed for selected
browser-facing services. Those later records do not prove current live state.

## Network Baseline

Primary LAN:

```text
Private LAN Subnet
```

Known LAN IP:

```text
Reserved LAN Address
```

Known Tailscale IP:

```text
Tailscale Private Address
```

At the time of the review, Tailscale was the private management plane for the
system.

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
Anywhere                    ALLOW IN    private LAN subnet
Anywhere (v6) on tailscale0 ALLOW IN    Anywhere (v6)
```

At the time of the review, this meant the system allowed:

* Traffic over the Tailscale interface
* Traffic from the trusted local LAN
* Outbound traffic from the host

It denies unsolicited incoming traffic by default outside those allowed paths.

## Tailscale Role

Tailscale provided private network access to `basement-node` in the 2026-06-16
baseline and remains documented as the private administrative path in the
later catch-up record.

Tailscale identity recorded during that review:

```text
private Tailscale address  basement-node
```

Operational role:

* Private remote management access
* Safer alternative to opening management ports publicly
* Useful path for reaching services while away from the local LAN
* Foundation for controlled administrative access

Later project history records Tailscale SSH as validated for emergency remote
administration. SFTP over SSH was also validated. Their current live status
still requires a new check.

## Docker Service Exposure History

**HISTORICAL STATE — 2026-06-16:** the monitoring stack published these ports
on the host:

```text
Grafana             3000/tcp
Prometheus          9090/tcp
cAdvisor            8080/tcp
node-exporter       9100/tcp
blackbox-exporter   9115/tcp
```

These ports were intended for trusted LAN and private-access use.

They were not intended as public internet services.

**LATER DOCUMENTED STATE:** unnecessary host-published ports were removed for
Prometheus, node-exporter, cAdvisor, and blackbox-exporter. Grafana remained
intentionally reachable as the primary monitoring UI, and internal monitoring
health checks passed after the exposure reduction. The tracked documentation
does not establish Grafana's exact current host binding or access path, so this
document does not infer a current port mapping.

Docker-published ports form a separate exposure boundary from ordinary host
listeners. Docker forwarding and NAT behavior can allow published container
ports to traverse paths that do not match expected UFW host filtering. This
does not mean that UFW "does not work with Docker"; it means every published
port needs deliberate binding, routing, and reachability review.

## Interactive Access Reliability

Interactive-access reliability is tracked separately from firewall and
identity boundaries.

The current catch-up record covers Bluetooth reconnect behavior,
`x11vnc-local.service` stability, and Fire Stick/Moonlight decoder behavior.
Each remains **OBSERVATION PENDING** and should not be described as
definitively resolved.

See [Interactive Access Observations](interactive-access-observations.md).

## Security Posture History

Strengths observed during the 2026-06-16 review:

* UFW is active
* Default incoming traffic is denied
* Tailscale is available for private access
* LAN access is explicitly scoped to `private LAN subnet`
* Services were not being intentionally exposed directly to the public internet
* Private bulk storage was unmounted during documentation/review preparation to reduce accidental data exposure

Limitations observed during the 2026-06-16 review:

* Some services publish ports on all host interfaces
* Grafana and Prometheus were suitable for trusted/private network access, not direct public exposure
* Service-level authentication and reverse proxy rules should be reviewed before broader access
* External browser access had not yet been placed behind Cloudflare Access

Later documentation records Cloudflare Tunnel as deployed, Cloudflare Access
as enforced for selected browser-facing access, and the monitoring port
reduction described above. These later facts supersede the corresponding
planning and exposure statements for the documented catch-up baseline, but
none has been revalidated against the live system during this documentation catch-up.

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
3. Confirm UFW allows LAN traffic from `private LAN subnet`.
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

## Historical Cloudflare Tunnel and Access Roadmap

The following section preserves the access plan as it was written. The dated
[Secure Remote Access and Custom Domain Email](secure-remote-access-and-domain-email.md)
record documents that this work was subsequently completed for its observation
period. Current live state still requires validation.

At the time, the proposed improvement was to add Cloudflare Tunnel and
Cloudflare Access for identity-aware browser access.

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

## Later Documented Access State

The roadmap above was later superseded by these documented results:

* Cloudflare Tunnel was deployed.
* Cloudflare Access was deployed for selected browser-facing remote access.
* The browser-desktop path uses Cloudflare Access/Tunnel, noVNC/websockify,
  and x11vnc.
* Nextcloud was deployed and made publicly reachable through Cloudflare
  Tunnel, with its local web origin bound to localhost.
* Tailscale SSH was validated for emergency remote administration.
* SFTP over SSH was validated for file transfer.

The exact current Cloudflare applications, policies, origins, DNS records,
tunnel health, and Grafana route require live validation. Do not inspect or
publish `cloudflared` service-unit contents: the deployment record indicates
that authentication material is embedded there.

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

* Revalidate which Docker services need host-published ports
* Keep internal-only services on Docker networks or suitable loopback/private bindings
* Revalidate Cloudflare Tunnel and Access policy enforcement without exposing connector authentication material
* Confirm and document Grafana's intended reachability and access-control path
* Document service-specific access requirements
* Add a simple access runbook
* Review Docker forwarding policy and whether narrower host/LAN rules are appropriate

## Summary

The 2026-06-16 baseline used a simple private-first access model: trusted LAN
access, Tailscale private remote access, and UFW deny-incoming by default.

Later documentation records Cloudflare Tunnel and Cloudflare Access for
selected browser-facing services, validated Tailscale SSH and SFTP paths, and
reduced host publication of internal monitoring ports. Current live state and
Grafana's exact reachability remain validation items.
