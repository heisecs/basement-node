# Hardware Upgrade and Validation Workflow

## Purpose

This document records a hardware upgrade and validation workflow performed on `basement-node`.

The goal was to increase host capacity while treating the system like a persistent infrastructure node: make the change, validate hardware detection, test stability, preserve a rollback point, observe service health, and document the result.

This was not treated as a casual desktop upgrade. It was handled as an infrastructure change with validation and recovery awareness.

## Historical Context

Status: **HISTORICAL STATE — June 2026**

This document preserves the June 2026 RAM-upgrade milestone. Its statements that the system had 32 GB describe the validated state at that time and were correct when recorded.

That memory baseline was later superseded by a 64 GB configuration. See [Current System Snapshot](current-system-snapshot.md) for the later documented baseline. The subsequent PSU and GPU milestone is recorded separately in [Power and R9700 Upgrade Validation](power-and-r9700-upgrade-validation.md); its full details are not duplicated here.

## Change Summary

`basement-node` was upgraded from 16 GB RAM to 32 GB RAM.

System context:

* Hostname: `basement-node`
* Operating system: Pop!_OS 24.04 LTS
* Hardware platform: MACHINIST X99 PR9-H
* RAM after upgrade: 32 GB installed
* Linux-visible memory: approximately 31 GiB
* RAM speed: 2133 MT/s
* Root storage: NVMe
* Recovery tool: Timeshift
* Monitoring stack: Dockerized Prometheus/Grafana stack

RAM was intentionally left at the default 2133 MT/s rather than enabling XMP. The priority was stability and predictable operation, not maximum theoretical memory speed.

## Firmware / BIOS Validation

After installation, the system firmware detected:

```text
32768 MB
```

The first boot after the RAM installation took longer than usual before video output appeared. This was likely memory training behavior. After a controlled power-button restart, the system entered BIOS successfully and the installed memory was visible.

## Operating System Validation

Memory visibility was checked from Pop!_OS using:

```fish
free -h
```

Observed result:

```text
Mem: 31Gi total
Swap: 19Gi total
```

This confirmed that the operating system could see the upgraded memory capacity.

## Stability Testing

A memory stress test was performed using:

```fish
stress-ng --vm 2 --vm-bytes 75% --timeout 10m --metrics-brief
```

Observed result:

```text
passed: 2: vm (2)
failed: 0
successful run completed in 10 mins
```

The test completed successfully with zero failures.

## Backup and Rollback Step

After the RAM upgrade was validated, a known-good Timeshift snapshot was created.

Snapshot:

```text
2026-06-09_16-56-53
```

Description:

```text
Known good after RAM upgrade to 32GB
```

This created a clear rollback point after the hardware change.

Timeshift snapshot management was also reviewed. Old automatic snapshots were cleaned up to reduce storage pressure while retaining useful recovery points.

Retained known-good recovery points included:

```text
2026-05-19_21-13-42  Known good after Docker Tailscale UFW setup with trimmed excludes
2026-06-09_16-56-53  Known good after RAM upgrade to 32GB
```

## Tooling and Platform Notes

The MACHINIST X99 PR9-H board reports inconsistent SMBIOS memory details, including unreliable slot-level information.

Because of that, tools such as `dmidecode` and `lshw` were not treated as fully authoritative for memory slot reporting on this platform.

Validation was instead based on:

* BIOS-level memory detection
* Operating system memory visibility
* Successful boot behavior
* Stress testing
* Post-change service health
* Known-good snapshot creation

This is a practical example of adapting validation methods when platform firmware data is imperfect.

## Post-Change Service Validation

After the hardware upgrade, core system health was checked.

Docker monitoring containers were confirmed running:

```text
prometheus
grafana
blackbox-exporter
cadvisor
node-exporter
```

Failed system services were checked using:

```fish
systemctl --failed --no-pager
```

Observed result:

```text
0 loaded units listed
```

This confirmed that the system was healthy after the hardware change.

## Issue Observed

Timeshift briefly segfaulted after the snapshot was created, during cleanup/pruning.

The snapshot was verified afterward, and the Timeshift snapshot list remained healthy. Because the known-good snapshot existed and was visible after validation, the recovery point was treated as usable.

## Lessons Learned

* Hardware upgrades should be paired with explicit validation, not assumed successful after boot.
* Stability-first settings can be more appropriate than performance tuning for a persistent infrastructure node.
* Recovery points should be created after successful validation.
* Snapshot retention should be managed intentionally so backups remain useful and storage-efficient.
* Firmware/SMBIOS data is not always reliable on lower-cost or unusual platforms.
* Practical validation should combine firmware checks, OS checks, stress testing, service checks, and rollback planning.

## Result

The RAM upgrade was successful.

`basement-node` now has 32 GB installed memory, approximately 31 GiB visible to Linux, a successful memory stress test, confirmed post-change service health, and a known-good Timeshift snapshot after validation.
