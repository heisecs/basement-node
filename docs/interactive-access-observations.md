# Interactive Access Observations

## Purpose

This document records troubleshooting observations for interactive
access to `basement-node`.

These notes describe reported behavior and targeted remediation. They do not
establish current live-system state or definitive root causes.

## Status Terminology

* **CONFIRMED** — directly validated behavior with sufficient evidence.
* **OBSERVED** — behavior seen during a specific troubleshooting period.
* **OBSERVATION PENDING** — an improvement or working result that still needs
  repeated validation.
* **HISTORICAL STATE** — earlier behavior retained for context rather than
  presented as current state.

None of the three areas in this document is promoted to **CONFIRMED**.

## Bluetooth Reconnect and Resume Behavior

Status: **OBSERVATION PENDING — IMPROVED, NOT RESOLVED**

### Hardware Change

The old problematic Broadcom Bluetooth adapter was replaced.

The replacement adapter was identified as:

```text
ASUSTek Bluetooth Controller
USB ID 0b05:1bf6
```

### Observed Behavior

Reconnect and resume behavior improved significantly after the adapter change.

* Previous waits were roughly 10–20 seconds.
* Current waits, when the issue occurs, are closer to 3–5 seconds.
* Occasional timeout behavior remains.

The shorter delays are evidence of improvement, but the remaining timeouts mean
the issue should not be described as resolved. The observations do not
establish a definitive Bluetooth root cause.

## x11vnc Service Stability

Status: **OBSERVATION PENDING**

Service:

```text
x11vnc-local.service
```

### Historical Failure Behavior

Status: **HISTORICAL STATE**

The service previously showed:

* Repeated crashes
* Signal 11
* systemd status `2/INVALIDARGUMENT`
* Refused connections on `localhost:5900`

### Targeted Remediation

The targeted change was to append the following x11vnc argument:

```text
-noxfixes
```

After the service was restarted, it became active.

That result is encouraging but does not yet demonstrate long-term stability,
reboot persistence, or resolution of the earlier crashes. The service should
not be called definitively fixed.

## Fire Stick and Moonlight Behavior

Status: **OBSERVATION PENDING**

### Comparative Observations

During the observed Fire Stick disconnects:

* Host Sunshine remained healthy.
* Moonlight on a phone remained stable.
* The Fire Stick used an H.264 MediaTek decoder.
* Reported network drops remained at 0%.
* Incoming and rendering rates stayed around 60 FPS.
* Decode latency sometimes increased dramatically.

### Current Interpretation

The comparison between clients, zero reported network drops, and large decoder
latency increases points toward Fire Stick client decoding, output buffering,
or pacing rather than simple raw network throughput.

This is a working interpretation, not a final root-cause finding.

### Current Practical Profile

```text
Resolution: 1920x1080
Frame rate: 30 FPS
Codec: H.264
Bitrate: 10 Mbps
Latency preference: Prefer lowest latency
```

The profile is a practical operating choice while observation continues. It
should not be treated as proof that the underlying behavior is resolved.

## Validation Still Required

Future validation should determine:

* Whether Bluetooth improvements persist across repeated reconnect and resume
  cycles
* Whether the remaining Bluetooth timeouts follow a reproducible pattern
* Whether `x11vnc-local.service` remains stable over time and after reboot
* Whether the Fire Stick behavior recurs under the practical profile
* Whether decoder latency continues to rise without corresponding network drops
* Whether comparisons with another Moonlight client remain consistent

## Related Documentation

* [Access and Security](access-and-security.md)
* [Architecture Overview](architecture-overview.md)
* [Secure Remote Access and Custom Domain Email](secure-remote-access-and-domain-email.md)

## Summary

The Bluetooth adapter replacement substantially improved reconnect and resume
delays, the `-noxfixes` change was followed by an active x11vnc service, and
Fire Stick evidence currently points toward client-side decoding or output
pacing. All three findings remain **OBSERVATION PENDING**.
