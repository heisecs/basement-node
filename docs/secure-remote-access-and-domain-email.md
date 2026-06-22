# Secure Remote Access and Custom Domain Email

## Purpose

This document records the 2026-06-16 remote access and custom-domain email setup for `basement-node`.

The work established a Cloudflare Tunnel path to a localhost-only browser remote desktop service and configured `chrisheise.dev` for professional custom-domain email through Cloudflare Email Routing and Gmail send-as.

## Summary

Completed work:

* Created a Cloudflare Zero Trust environment.
* Installed `cloudflared` on `basement-node`.
* Connected `basement-node` to Cloudflare using a Cloudflare Tunnel connector token.
* Published the existing noVNC remote desktop endpoint through Cloudflare Tunnel.
* Confirmed browser-based remote desktop access to the KDE/X11 session.
* Purchased and configured `chrisheise.dev`.
* Configured Cloudflare Email Routing for inbound custom-domain email.
* Configured Gmail send-as / reply-as for the custom-domain address.

## System Context

* Hostname: `basement-node`
* Operating system: Pop!_OS 24.04 LTS
* Desktop: KDE Plasma on X11
* Primary user: `sona`
* Shell: fish
* `cloudflared` version: `2026.6.0`

Local remote desktop components:

* `x11vnc`
* `websockify`
* `noVNC`

## Local Service Baseline

Remote desktop services were confirmed listening on localhost only:

```text
127.0.0.1:5900  x11vnc
127.0.0.1:6080  websockify / noVNC
```

This binding model is intentional.

Local access model:

* `x11vnc` provides access to the active KDE/X11 desktop session.
* `websockify` / noVNC provides browser access to the VNC session.
* Both local services remain bound to `127.0.0.1`.
* Cloudflare Tunnel points to the local noVNC web endpoint.
* The VNC/noVNC password remains enabled as a second authentication layer.

Local noVNC service target:

```text
http://127.0.0.1:6080
```

Browser path:

```text
/vnc.html
```

## Remote Access Architecture

Remote desktop access now follows this path:

```text
Browser
  -> a protected remote-access subdomain
  -> Cloudflare
  -> Cloudflare Tunnel
  -> cloudflared on basement-node
  -> http://127.0.0.1:6080
  -> noVNC / websockify
  -> x11vnc
  -> KDE/X11 desktop session
```

This avoids direct router port forwarding for the remote desktop service.

`cloudflared` creates an outbound connection from `basement-node` to Cloudflare. Browser traffic reaches Cloudflare first and is relayed through the tunnel to the local noVNC endpoint.

## Cloudflare Tunnel Setup

Cloudflare Zero Trust was configured.

A Cloudflare Tunnel connector was installed on `basement-node` using the Cloudflare-provided connector command and token.

Observed tunnel state during setup:

```text
Connected
Healthy
```

The public hostname was configured to route to:

```text
http://127.0.0.1:6080
```

## Domain and DNS

Professional domain purchased:

```text
Protected Remote Access Domain
```

The domain was added to the Cloudflare workflow and used for the remote access hostname.

Remote access hostname:

```text
remote.<personal-domain>
```

Original issue observed with a different domain:

* Owning a domain was not enough for the hostname to resolve.
* The hostname must have valid DNS.
* For Cloudflare-managed hostnames, the domain must be active in Cloudflare DNS or have the required tunnel DNS record.

Expected Cloudflare Tunnel DNS pattern:

```text
remote CNAME <tunnel-uuid>.cfargotunnel.com
```

## Remote Desktop Validation

Remote desktop validation succeeded.

Confirmed result:

* The Cloudflare hostname loaded the noVNC page.
* noVNC connected through the tunnel.
* The KDE/X11 desktop session was visible in the browser.
* The desktop was controllable through the browser.
* The live VNC feedback effect confirmed access to the active desktop session.

Usability notes:

* In normal browser mode, the mouse can usually leave the noVNC canvas by moving or clicking outside it.
* In fullscreen or pointer-capture mode, press `Esc` to escape.
* `Alt+Tab`, `Ctrl+L`, or closing the browser tab can also recover control if needed.

## Access Control Status

Cloudflare Tunnel remote access is working.

Cloudflare Access protection has been  confirmed and this remote desktop path should be treated as fully protected for ongoing use.

Required control:

```text
remote.<personal-domain> -> Cloudflare Access policy -> Cloudflare Tunnel -> localhost noVNC endpoint
```

Required verification:

* Confirmed a Cloudflare Access self-hosted application exists for `remote.<personal-domain>`.
* Confirmed the policy allows only the intended owner email address.
* Confirmed unauthenticated access is blocked.
* Confirmed the VNC/noVNC password remains enabled.

## Custom Domain Email

Cloudflare Email Routing was configured for:

```text
chrisheise.dev
```

Public contact address:

```text
contact@chrisheise.dev
```

Mail flow:

```text
contact@chrisheise.dev
  -> Cloudflare Email Routing
  -> Gmail inbox
```

Result:

* Inbound custom-domain mail is forwarded to Gmail.
* Gmail receives mail sent to the custom-domain address.

## Gmail Send-As / Reply-As

Gmail was configured to send and reply using the custom-domain address.

Configured Gmail behavior:

* Added `contact@chrisheise.dev` under Gmail “Send mail as.”
* Confirmed replies can use the custom-domain identity.
* Gmail can compose new messages from the custom address using the `From` dropdown.

Gmail settings path:

```text
Gmail -> Settings -> See all settings -> Accounts and Import -> Send mail as
```

## Email Model

Current email model:

```text
Inbound mail:
sender -> contact@chrisheise.dev -> Cloudflare Email Routing -> Gmail

Outbound mail:
Gmail -> Send mail as contact@chrisheise.dev -> recipient
```

This provides custom-domain email without self-hosting a mail server.

Self-hosting email was intentionally deferred because reliable mail hosting requires additional infrastructure and deliverability controls beyond the current `basement-node` scope.

## Validation Completed

Validated during the session:

* `cloudflared` installed successfully.
* `cloudflared` version confirmed as `2026.6.0`.
* Cloudflare tunnel connector reached `Connected` / `Healthy` state.
* `x11vnc` remained bound to `127.0.0.1:5900`.
* `websockify` / noVNC remained bound to `127.0.0.1:6080`.
* Cloudflare public hostname reached the noVNC endpoint.
* Browser-based remote desktop connected to the KDE/X11 session.
* `chrisheise.dev` was purchased and configured.
* Cloudflare Email Routing was configured.
* Gmail send-as / reply-as was configured for `contact@chrisheise.dev`.

## Current Working State

Working now:

* `cloudflared` is installed.
* Cloudflare tunnel connector is healthy.
* noVNC remote desktop is reachable through Cloudflare Tunnel.
* Browser remote desktop has been confirmed working.
* `chrisheise.dev` is configured.
* `contact@chrisheise.dev` receives mail through Cloudflare Email Routing.
* Gmail can send/reply using `contact@chrisheise.dev`.
* Cloudflare Access protection is enforced for `remote.<private-domain>`.
* Confirm unauthenticated users cannot reach the noVNC page.
* `cloudflared` starts on boot.
* `x11vnc` and `websockify` startup persistence.
* Confirmed remote access from an off-network device.
* Confirmed sending a new composed email from `contact@chrisheise.dev`.
* Confirmed replying to inbound mail sent to `contact@chrisheise.dev`.

## Skills Practiced

* Cloudflare Zero Trust setup
* Cloudflare Tunnel connector deployment
* DNS and public hostname configuration
* Remote access without router port forwarding
* Localhost-only service binding
* Browser-based remote desktop with noVNC
* Layered remote access design
* Custom-domain email routing
* Gmail send-as / reply-as configuration
* Infrastructure validation and documentation

## Resume Bullet

Built browser-based remote desktop access to a Linux homelab using Cloudflare Tunnel, noVNC, websockify, and x11vnc with localhost-only service binding; configured custom-domain email using Cloudflare Email Routing and Gmail send-as integration.

## Summary

This session established a working Cloudflare Tunnel path to the localhost-only noVNC remote desktop service on `basement-node` and configured `chrisheise.dev` for custom-domain email.

