# Architecture Overview

## Purpose

`basement-node` is a dual-purpose Linux server and workstation. It hosts persistent self-managed services while retaining a local graphical desktop for administration and interactive workloads. This overview describes the documented architecture at a public-safe level; it is not a substitute for live-state verification or production configuration.

## Architecture at a Glance

```text
Local network                         Remote users and administrators
  |                                      |                 |
  | SSH / SFTP / Moonlight               | Tailscale       | Cloudflare Access
  |                                      | private admin   | protected web access
  +-------------------+------------------+-----------------+
                      |
              basement-node
       Pop!_OS + KDE Plasma on X11
                      |
       +--------------+---------------+
       |              |               |
   Host services   Docker stacks   Local storage
   SSH, Sunshine   Monitoring      NVMe system volume
   noVNC/x11vnc    Nextcloud       ext4 bulk/data volumes
                      |
       exporters -> Prometheus -> Grafana
```

Cloudflare and Tailscale provide different access boundaries: Cloudflare fronts selected browser-accessible services through identity-aware access and outbound tunnels, while Tailscale provides a private network path for administration.

## Host and Session Layer

The host runs Pop!_OS 24.04 LTS and serves both infrastructure and workstation roles. KDE Plasma on X11 is the graphical-session baseline. Host-level services provide secure shell access, file transfer over SSH, graphical streaming, and the local endpoints used by browser-based remote desktop access.

This dual role means graphical-session changes can affect remote interactive access, while host or container changes can affect persistent services. Those concerns share hardware but should be operated and validated separately.

## Storage Architecture

The storage design separates the operating system from larger service and application data:

- An NVMe root volume holds the operating system and applications.
- Separate ext4 volumes hold bulk application, media, and service data.
- Nextcloud data is placed on secondary bulk storage rather than the root volume.
- Monitoring data is managed within the Docker monitoring stack.

Timeshift snapshots protect the local system state for rollback. They are stored locally and are not an independent backup of application data or protection against storage-device failure. In particular, Nextcloud still requires tested backup and restore procedures before it can be treated as part of a complete recovery architecture.

## Docker and Service Stacks

Docker Compose organizes persistent applications into service stacks rather than installing every component directly on the host. The documented architecture has two notable stack boundaries:

- The Docker Compose monitoring stack runs five containers: Prometheus, Grafana, node-exporter, cAdvisor, and blackbox-exporter.
- The Nextcloud stack provides the self-hosted cloud application and keeps its persistent data on bulk storage.

The stacks remain operationally separate even though they share the host, Docker runtime, storage, access controls, and recovery considerations. Internal components should remain internally reachable where possible; only required user-facing endpoints should be published or tunneled.

## Monitoring Architecture

The monitoring data flow is:

```text
Linux host metrics --------> node-exporter --+
Docker/container metrics --> cAdvisor -------+--> Prometheus --> Grafana
Endpoint reachability -----> blackbox-exporter+
```

Prometheus collects time-series data, Grafana is the primary visualization interface, node-exporter represents the Linux host, cAdvisor represents container workloads, and blackbox-exporter checks endpoint reachability. Host-published ports for internal monitoring components have been reduced; Grafana remains the main monitoring UI.

## Administration and Remote Access

The architecture supports several access paths with different purposes:

- LAN SSH is the preferred administrative path when on the trusted local network.
- Tailscale SSH provides private remote administration without making SSH a public internet service.
- SFTP uses the SSH path for controlled file transfer.
- Sunshine on the host and Moonlight on a client provide an interactive graphical streaming path. This is distinct from the browser-based access path and is currently subject to documented capture degradation under KDE/X11.
- Cloudflare Access and Tunnel provide identity-gated browser access to selected services. The remote desktop path terminates at a local noVNC/websockify endpoint, which connects through x11vnc to the KDE/X11 session.

Cloudflare is therefore the protected web-publishing layer; Tailscale is the private administrative network. Neither replaces the other.

## Nextcloud Placement

Nextcloud is a deployed Docker Compose workload, not a future placeholder. Its local web origin is bound to localhost and published through Cloudflare Tunnel rather than a directly exposed inbound port. Its data resides on secondary bulk storage.

Architecturally, Nextcloud sits at the intersection of application hosting, persistent storage, identity-aware web access, monitoring, and backup design. Deployment and phone upload have been validated, but tested backup and restore automation remains future work.

## Security Boundaries

UFW provides a default-deny inbound host policy, with trusted local and private administrative paths allowed intentionally. Selected browser-accessible services use outbound Cloudflare tunnels, avoiding direct public inbound routing to their local origins.

Docker networking is a separate boundary: published container ports can traverse Docker forwarding rules in ways that do not match expected UFW host filtering. The architecture therefore relies on minimizing published ports, keeping internal services internal, binding suitable origins to localhost, and exposing only intentional entry points.

No public architecture document should contain tunnel tokens, credentials, private keys, raw production configuration, or unnecessary private topology.

## Rollback and Recovery

Timeshift provides local rollback points before high-risk host, graphics, and platform changes. The operating model is to preserve a known-good state, make controlled changes, validate host and service behavior, and retain a usable rollback path.

This is a system-recovery layer, not a complete data-protection strategy. Persistent Docker application data—especially Nextcloud data—needs independent, tested backup and restore coverage.

## Future Local-AI Expansion

The AMD Radeon AI PRO R9700 with 32 GB of VRAM is a planned expansion and is **not installed**. The RX 6600 XT remains the active GPU in the documented baseline.

The R9700 is intended to add higher-memory GPU capacity for future local-AI and compute experimentation. No R9700-backed local-AI service stack is documented as installed or operational. Any future integration should be treated as a separate platform change with power, driver, graphical-session, remote-access, monitoring, and rollback validation.
