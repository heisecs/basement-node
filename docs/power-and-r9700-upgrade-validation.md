# Power Supply and R9700 Upgrade Validation

## Purpose

This document records the power-supply migration and AMD Radeon AI PRO R9700 installation on `basement-node`.

It preserves the former RX 6600 XT configuration as historical state, records the staged upgrade procedure, and distinguishes validated hardware behavior from known non-fatal log warnings.

This record is based on the project-history handoff. No new live-system inspection was performed for this documentation catch-up.

## Change Summary

Status: **CONFIRMED**

Installed power supply:

```text
EVGA SuperNOVA 850 P6 Platinum
Model: 220-P6-0850-X1
```

Current active GPU:

```text
Gigabyte AMD Radeon AI PRO R9700 AI TOP 32G
32 GB GDDR6
```

The upgrade established a validated higher-memory GPU platform for containerized ROCm and local-AI workloads.

## Historical Graphics State

Status: **HISTORICAL STATE**

Before the R9700 installation, the active GPU was:

```text
XFX Radeon RX 6600 XT 8 GB
```

The RX 6600 XT was the known-good pre-R9700 graphics baseline. It is no longer the current active GPU.

## Power-Supply Installation

Status: **CONFIRMED**

The R9700 power path uses:

- Two separate original EVGA VGA/PCIe cables
- The PSU's `VGA1` and `VGA2` connections
- No daisy-chained PCIe power connection
- No mixed modular PSU cables

Modular PSU cables are not assumed to be interchangeable, even when connectors appear physically compatible. Only the original EVGA cables associated with the installed PSU were used.

## Staged Upgrade Validation

Status: **CONFIRMED**

The PSU migration and GPU replacement were separated into stages:

1. Install the EVGA SuperNOVA 850 P6.
2. Retain and validate the RX 6600 XT after the PSU migration.
3. Install the R9700 only after the PSU migration had a known-good graphics result.
4. Validate the R9700 at the operating-system, graphics, and device-access layers before beginning ROCm compute validation.

This preserved a narrower troubleshooting boundary between power-supply migration and GPU installation.

## R9700 Validation

Status: **CONFIRMED**

Validated hardware and driver properties:

```text
GPU: Gigabyte AMD Radeon AI PRO R9700 AI TOP 32G
Memory type/capacity: 32 GB GDDR6
Reported VRAM: 32768 MB
GPU architecture: gfx1201
Compute units: 64
Kernel driver: amdgpu
OpenGL acceleration: active
OpenGL version: 4.6
PCIe link: Gen3 x16 on the current X99 platform
```

The Gen3 link is the validated operating state on the current X99 platform. It should not be rewritten as a newer platform capability.

## GPU Device and Group Access

Status: **CONFIRMED**

Required device interfaces exist:

```text
/dev/kfd
/dev/dri/renderD128
```

The required `video` and `render` group access is present.

Observed Docker group IDs on this host:

```text
video  = 44
render = 992
```

These numeric group IDs are host-specific and should not be treated as portable defaults for another machine.

## Known Non-Fatal Boot and Log Breadcrumbs

Status: **KNOWN NON-FATAL WARNING**

Observed R9700-related breadcrumbs:

- An SMU interface mismatch warning appears while initialization still completes.
- Two `REG_WAIT` timeout messages involve `optc401`.
- A MES firmware warning refers to the LR compute workaround.

These messages are retained as troubleshooting breadcrumbs.

They are not documented as fixed, as hardware defects, or as causes of unrelated failures. Their presence does not override the successful graphics and device-access validation recorded in this document. Later ROCm/HIP/PyTorch compute validation is documented separately in `docs/rocm-and-llama-cpp-stack.md`.

## Rollback Point

Status: **CONFIRMED**

Known-good Timeshift snapshot:

```text
2026-08-17_00-12-34
```

Description:

```text
Known-good post-R9700 + EVGA 850 P6 install, pre-ROCm
```

This snapshot captures the validated PSU and GPU state before the containerized ROCm work. It provides a useful separation between hardware integration and later AI userspace/application changes.

Timeshift remains a local system rollback mechanism rather than an independent data-backup system.

## Result

The EVGA SuperNOVA 850 P6 and AMD Radeon AI PRO R9700 upgrades were successfully validated.

The current documented GPU baseline is the R9700 running through `amdgpu`, with accelerated OpenGL, 32 GB of reported VRAM, `gfx1201` discovery, 64 compute units, PCIe Gen3 x16 operation on the current X99 platform, and the device/group access required by containerized ROCm workloads.

The RX 6600 XT remains documented as the pre-R9700 historical baseline.
