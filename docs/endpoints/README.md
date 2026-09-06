# Endpoint and Client Devices

## Purpose

This directory documents endpoint and client-device work connected to
`basement-node`.

The focus is on devices that interact with the server as thin clients,
streaming clients, automation controllers, remote-access tools, sideloaded
clients, VR endpoints, or experimental application/runtime clients.

These documents preserve engineering decisions, validation results,
integration behavior, and open issues. They are not intended to expose private
device identifiers, credentials, secrets, raw personal data, or device-specific
recovery artifacts.

## Scope

Endpoint documentation may include:

* Android phones and tablets used as thin clients or automation surfaces
* Sunshine/Moonlight clients
* Sideloaded Android-derived devices
* VR headsets that integrate with server-hosted applications or streaming
  workflows
* Controller and input-device behavior tied to endpoint use
* Device-specific networking, latency, decoder, display, and power observations
* Security boundaries for rooted, sideloaded, or repurposed devices

## Current Structure

```text
docs/endpoints/
├── README.md
└── android/
    └── galaxy-s5-sm-g900v.md
```

The `android/` directory groups the first documented Android endpoint. It does
not require every Android-derived device to use the same category. A future
Quest or other VR record may fit better under a functional VR category once
source-backed project history exists.

## Current Endpoint Documents

### Galaxy S5 SM-G900V

Status: **CONFIRMED — SUCCESSFUL PLATFORM CONVERSION**

Document: [Galaxy S5 SM-G900V Android Thin Client](android/galaxy-s5-sm-g900v.md)

A Verizon Samsung Galaxy S5 SM-G900V was converted from stock carrier Android
6.0.1 into a bootloader-unlocked, LineageOS 18.1, Magisk-rooted, de-Googled
Android thin client for `basement-node`.

The project included variant and CID-path verification, live PIT inspection,
recovery-path preservation, historical-tooling analysis, a controlled
temporary-root and bootloader-unlock workflow, Download Mode validation,
standalone TWRP, a private disaster-recovery backup, exact-ROM Magisk boot-image
handling, Linux-host ADB/udev integration, F-Droid installation, and
Moonlight/Sunshine validation.

The bootloader, ROM, root, and baseline thin-client phase is complete. Appliance
integration and long-term usability validation remain open.

## Planned Endpoint Areas

The following records are planned only after source-backed project histories
are available. No device-specific document should imply that work has already
been performed.

### Galaxy Tab S7

Status: **PLANNED / BACKLOG**

Expected scope includes Winlator experiments, Android gaming compatibility,
controller testing, and server/client integration notes.

### Galaxy Tab S2

Status: **PLANNED / BACKLOG**

Expected scope includes modernization or repurpose planning, lightweight
endpoint use, and practical device limitations.

### Galaxy S25 Ultra

Status: **PLANNED / BACKLOG**

Expected scope includes Tasker automation, a Ford steering-wheel-button
voice-agent/control workflow, the phone-to-server control path, and explicit
safety boundaries for driving-mode use.

### Fire TV Stick

Status: **OBSERVATION PENDING / PLANNED DOCUMENTATION**

Expected scope includes its Sunshine/Moonlight profile, decoder and latency
behavior, and the distinction between host-side Sunshine health and client-side
decode or output behavior. Some current Fire Stick observations are represented
in [Interactive Access Observations](../interactive-access-observations.md), but
the full endpoint history has not yet been consolidated into the repository.

### Quest Headset

Status: **PLANNED / BACKLOG**

Expected scope includes sideloading, Android-derived endpoint behavior,
VR/server integration, and server-linked VR workflows where applicable. Its
eventual location should follow the documented function rather than being
assumed from its Android-derived platform.

## Status Terminology

Use the following status language consistently:

* **CONFIRMED** — a result was directly validated and is suitable to treat as
  known working state for the recorded project period.
* **OBSERVED** — a behavior was seen during testing but has not been repeated
  enough to support a durable conclusion.
* **OBSERVATION PENDING** — a behavior improved, changed, or gained a likely
  explanation, but needs repeated validation before being called resolved.
* **HISTORICAL STATE** — a statement was correct when recorded, but later work
  changed the documented baseline.
* **PLANNED / BACKLOG** — a task, validation step, or document has not yet been
  completed.

Repository documentation remains separate from live state. A recorded endpoint
result does not prove that the device, server path, application, or network
integration is still operating unchanged.

## Public Documentation Safety

Do not publish or commit:

* IMEI values or serial numbers
* Full raw CID values
* ADB serial values
* Raw EFS, modem, or other device-identity data
* Credentials, private keys, recovery codes, or authentication material
* Wi-Fi secrets or private network details
* Recovery images, private binary backups, or proprietary backup archives
* Raw logs containing sensitive identifiers
* Private personal media or unnecessary device inventories

Acceptable material, when technically useful, includes:

* Public device model and variant
* Android and ROM versions
* High-level recovery and root state
* Sanitized command examples
* Public tool names and provenance notes
* Checksums for public downloads when their provenance is clear
* High-level backup categories
* Engineering decisions and validation outcomes

Treat hashes of private device-specific artifacts cautiously. Public-tool
checksums can support provenance, but private recovery artifacts and their
metadata do not belong in the public repository.

## Relationship to basement-node

An endpoint belongs here when it materially participates in the
`basement-node` environment. Examples include:

* Acting as a Moonlight client for the Sunshine host
* Providing remote administration or control
* Validating a server-hosted application path
* Using a self-hosted service
* Becoming a repeatable thin-client or appliance endpoint
* Demonstrating relevant security, networking, input, display, decoder, or
  latency behavior

Personal device customization without a meaningful `basement-node` connection
should remain outside the main infrastructure documentation.

## Related Infrastructure Documentation

* [Architecture Overview](../architecture-overview.md)
* [Access and Security](../access-and-security.md)
* [Current System Snapshot](../current-system-snapshot.md)
* [Interactive Access Observations](../interactive-access-observations.md)
* [Roadmap](../roadmap.md)
