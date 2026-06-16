# Backup and Recovery

## Purpose

This document describes the current backup and recovery posture for `basement-node`.

The goal is to keep the system recoverable while making changes, upgrades, and service adjustments. This is especially important because `basement-node` is treated as a persistent infrastructure node rather than a disposable test machine.

## Recovery Tool

`basement-node` currently uses Timeshift in RSYNC mode for system snapshots.

Timeshift is used to create restore points before or after important system changes, including:

* Docker/service setup
* Firewall/access changes
* Hardware upgrades
* Documentation and review preparation
* Known-good system states

## Current Timeshift State

Observed Timeshift status during the 2026-06-16 system review:

```text
Device : /dev/nvme0n1p3
Mode   : RSYNC
Status : OK
10 snapshots, 753.3 GB free
```

Timeshift is using the root NVMe filesystem as the snapshot target.

## Important Recovery Points

Current important snapshots:

```text
2026-05-19_21-13-42  Known good after Docker Tailscale UFW setup with trimmed excludes
2026-06-09_16-56-53  Known good after RAM upgrade to 32GB
2026-06-16_14-45-51  pre-demo-basement-node-portfolio-prep
```

These snapshots represent useful rollback points:

* A known-good service/access baseline
* A known-good post-hardware-upgrade baseline
* A pre-review/pre-documentation preparation baseline

## Snapshot Strategy

The current strategy is to retain known-good snapshots around meaningful infrastructure changes instead of relying only on automatic snapshots.

Useful snapshot timing:

* Before risky configuration changes
* After successful hardware upgrades
* After successful service deployments
* Before major cleanup or refactoring
* Before access/security changes
* Before documentation or review sessions where accidental changes are possible

## Exclusions and Storage Management

Large non-system paths are excluded from Timeshift where appropriate.

The purpose is to keep snapshots focused on system recoverability rather than copying large datasets, caches, or personal-use storage.

This keeps snapshots more manageable and reduces the chance that recovery storage becomes noisy or wasteful.

## Validation Commands

List snapshots:

```fish
sudo timeshift --list
```

Create a manual snapshot:

```fish
sudo timeshift --create --comments "describe-change-here"
```

Check root filesystem space:

```fish
df -h /
```

Check mounted filesystems:

```fish
lsblk -f
```

## Current Storage / Privacy Note

During the 2026-06-16 documentation and review preparation session, the private bulk-storage volume was cleanly unmounted.

Observed state after unmount:

```text
sda2  ext4  media-primary  no mountpoint
```

The secondary storage volume remained mounted:

```text
sdb2  ext4  media-secondary  /mnt/media-secondary
```

This reduces accidental exposure of data that is not relevant to the infrastructure work being reviewed or documented.

## Recovery Mindset

The operating model for `basement-node` is:

1. Understand the current state.
2. Make one meaningful change at a time.
3. Validate the result.
4. Create or retain a rollback point.
5. Document what changed and why.
6. Avoid mixing unrelated troubleshooting threads when possible.

This makes the system easier to recover, explain, and continue improving.

## Known Notes

During a previous snapshot management session, Timeshift briefly segfaulted after a snapshot had already been created during cleanup/pruning.

The snapshot list was checked afterward and remained healthy. The known-good snapshot was visible after validation, so it was treated as usable.

This is a useful reminder to verify backups after creating or cleaning them rather than assuming success.

## Future Improvements

Planned backup/recovery improvements:

* Keep a clearer snapshot naming convention
* Document which paths are excluded from Timeshift
* Add a recovery runbook
* Add a pre-change checklist
* Add a post-change validation checklist
* Consider separate backup strategy for non-system data
* Periodically test restore assumptions
* Avoid letting automatic snapshots crowd out meaningful known-good restore points

## Summary

Timeshift is part of the operating model for `basement-node`.

The system currently has multiple useful restore points, including a known-good post-RAM-upgrade snapshot and a pre-review preparation snapshot. Backup and recovery are treated as active infrastructure practices, not afterthoughts.

