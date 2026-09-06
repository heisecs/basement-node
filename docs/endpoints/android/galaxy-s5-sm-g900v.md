# Galaxy S5 SM-G900V Android Thin Client

## Purpose

This document records the Verizon Galaxy S5 SM-G900V endpoint conversion
project.

The goal was to convert a legacy carrier Android phone into a usable
Android-based thin client for the `basement-node` environment, focused on
de-Googled operation, Moonlight/Sunshine streaming, Linux-host integration, and
durable recovery paths.

This is an endpoint/client-device record, not a generic Android modding guide.
It is based on the completed project handoff; no new live-device or live-server
inspection was performed while preparing this repository document.

## Summary

Status: **CONFIRMED — SUCCESSFUL PLATFORM CONVERSION**

Confirmed final project state:

```text
Device: Verizon Samsung Galaxy S5
Model: SM-G900V
Lineage device family: klte
Installed device property: kltevzw
Android: 11
ROM: LineageOS 18.1
Build: lineage-18.1-20240306-nightly-klte-signed.zip
Recovery: standalone TWRP 3.3.1-0
Root: Magisk 30.7
Google Apps: not installed
F-Droid: installed and functional
Moonlight: installed and working
Sunshine discovery over LAN: functional
Moonlight pairing: successful
ADB on LineageOS: functional
```

Download Mode reported the expected custom-binary and developer-mode indicators
after the unlock workflow. Magisk root was validated from ADB with a root UID
and the expected Magisk SELinux context. Device-specific identifiers are
intentionally omitted.

The difficult bootloader, ROM, root, recovery, and baseline thin-client phase is
complete. Remaining work concerns appliance integration and long-term usability.

## System Role

The Galaxy S5 is being repurposed as a small Android endpoint for
`basement-node`:

```text
basement-node
→ Sunshine host
→ Moonlight on the Galaxy S5
→ de-Googled Android thin client
```

The project validates an endpoint pattern relevant to the infrastructure lab:

* Repurpose legacy hardware as a narrow thin-client appliance.
* Gate destructive changes on exact variant and storage-path identification.
* Preserve recovery paths before bootloader, firmware, ROM, or root changes.
* Integrate Android ADB permissions with the Linux host.
* Validate the server/client path rather than only the device software.
* Defer permanent installation until display, input, power, and thermal behavior
  are tested.

## Original Device Baseline

Status: **HISTORICAL STATE**

The starting software state was:

```text
Android: 6.0.1
Build: MMB29M.G900VVRU2DPG2
Bootloader: G900VVRU2DPG2
Binary and system status: Samsung Official
Developer Mode: not present
```

Download Mode also reported Qualcomm Secure Boot and Secure Download as
enabled. Reactivation Lock was off.

The phone had reportedly spent most of roughly a decade unused. Under a one-hour
YouTube playback observation, charge fell from 74% to 8% in a smooth progression.
The device remained cool, did not show sudden battery collapse, and had no
observed swelling.

Status: **OBSERVED**

The aged battery was adequate for controlled testing but has substantially
degraded usable capacity. Battery replacement or another safe long-term power
strategy should precede permanent appliance installation.

## Device Identification and CID Gate

Status: **CONFIRMED**

The exact Verizon variant was treated as a gating condition rather than relying
on generic Galaxy S5 instructions.

The relevant CID began with `15`, identifying the Samsung eMMC path compatible
with the historical CID15 SamsungCID bootloader-unlock workflow. The complete
CID is intentionally not recorded because the technical decision depends on the
CID class, not the full device-specific value.

This check was essential: Verizon SM-G900V bootloader procedures differ from
procedures for globally unlockable Galaxy S5 variants.

## PIT and Partition Validation

Status: **CONFIRMED**

A live PIT was obtained with Heimdall before flashing:

```fish
heimdall print-pit --no-reboot
```

The device reported the `MSM8974.pit` layout with approximately 30 entries.
Relevant mappings were checked for the bootloader, radio, boot, recovery,
system, cache, persist, and personal-data partitions.

Engineering decision:

* Do not blindly repartition the phone.
* Do not flash a supplied PIT unless the workflow requires it.

The live layout appeared coherent, and the selected unlock workflow did not
require repartitioning. This avoided adding partition-table risk to an already
high-risk firmware transition.

## Recovery and Provenance Preparation

Status: **CONFIRMED**

Stock DPD1 firmware was downloaded and checksum-verified as fallback material,
but it was not used as the successful primary route. It remains historical
recovery material rather than a required step in the completed workflow.

Multiple historical Verizon S5 unlock packages were also inspected before
execution. That review was necessary because the ecosystem contains obsolete
backend dependencies, device-specific assumptions, repackaged tools, and
low-level binaries that should not be trusted solely from filenames or forum
instructions.

Provenance rules used during the project:

* Prefer a published checksum when one is available.
* Treat a locally calculated hash without an independent published value as
  local provenance only.
* Inspect updater scripts and bundled executables before use.
* Validate the result from device state instead of assuming a script succeeded.
* Keep private recovery artifacts outside the public repository.

## Rejected Unlock and Root Routes

### SafeStrap PG2 Bootloader Unlock AIO

Status: **HISTORICAL STATE — INVESTIGATED, NOT USED**

The package updater was inspected. It would flash the radio, boot, recovery,
and bootloader chain before invoking a SamsungCID-style unlock binary. Static
strings showed eMMC/CID operations, bootloader backup behavior, and microSD
requirements.

The package helped explain the historical unlock method, but it was not used
for the final unlock.

### Original and Modified Towelroot

Status: **HISTORICAL STATE — REJECTED ROUTE**

The original Towelroot APK installed on the engineering firmware but continued
to demand internet connectivity after connectivity and device time were
checked. Investigation of original and modified variants indicated obsolete
backend dependencies.

The route was abandoned rather than weakening network or device controls to
support an obsolete service dependency.

### 2022 Root Tool Bundle

Status: **CONFIRMED SOURCE MATERIAL — LOCAL PROVENANCE ONLY**

A later root-tool bundle contained the components needed for the successful
temporary-root, SuperSU, Safestrap, and CID15 workflow. No independently
published checksum was found for the complete bundle, so its local hash was not
treated as third-party verification.

Static inspection established the likely sequence:

```text
APA1 engineering firmware
→ KingRoot/schrodinger32 temporary root
→ replace KingRoot with SuperSU
→ install Safestrap
→ apply the CID15 QL1 unlock package
→ validate unlock state in Download Mode
```

Static analysis informed the execution plan. It did not replace observed device
state as the final validation authority.

## Combination Firmware

Status: **CONFIRMED**

The exact engineering-firmware package used was:

```text
COMBINATION_VZW_FA44_G900VVRU2APA1_VZW2APA1_2572656_REV00_user_mid_noship_MULTI_CERT.tar.md5
```

The package supplied the expected bootloader, radio, boot, recovery, system,
persist, personal-data, and cache images. Its bundled PIT was deliberately not
flashed because the live PIT did not show a need for repartitioning.

Heimdall flashed explicit partition mappings successfully. The phone then
booted Samsung engineering/factory software, and ADB worked with an unprivileged
shell identity.

## Temporary Root and SuperSU Transition

Status: **CONFIRMED**

KingRoot 5.4.0 and the inspected `schrodinger32` chain established temporary
root on the APA1 engineering firmware. The staged exploit ended with its
reported root-success marker, and ADB validation returned a root UID.

The inspected replacement script then removed KingRoot and installed SuperSU
2.82-SR5. After reboot and explicit approval of the SuperSU prompt, root was
again validated from ADB.

Temporary proprietary root tooling was therefore used as a transition, not as
the final rooted operating state.

## Safestrap Transition

Status: **CONFIRMED — TRANSITIONAL LIMITATION**

Safestrap installed its recovery environment, reporting Safestrap 4.11 with
TWRP 3.3.1-0 on the stock ROM slot.

ADB detected the recovery and file transfer worked, but an interactive recovery
shell failed because required runtime libraries could not be linked. The broken
shell was accepted as a temporary limitation because staging still worked and
standalone TWRP would replace Safestrap after bootloader unlock.

## CID15 Bootloader Unlock

Status: **CONFIRMED**

The inspected CID15 QL1 package was staged and installed through the Safestrap
recovery environment. Its updater replaced the system and required firmware
chain, removed the Safestrap filesystem-check hijack, restored normal filesystem
handling, and invoked the SamsungCID unlock component.

After installation, the QL1 system booted. Download Mode then reported:

```text
CURRENT BINARY: Custom
SYSTEM STATUS: Custom
MODE: Developer
```

Those indicators were used as the authoritative bootloader-unlock result.

No second SamsungCID pass was performed. Repeating a low-level bootloader
operation after Download Mode showed the desired state would add risk without
adding useful confidence.

## Standalone TWRP

Status: **CONFIRMED**

Standalone TWRP 3.3.1-0 was extracted from the verified CID15 package and
flashed to the recovery partition with Heimdall using `--no-reboot`.

During this workflow, stock Android was intentionally prevented from booting
before the first standalone-TWRP boot because affected Samsung stock
configurations can restore stock recovery. This was a precaution against a
known platform risk; the project did not observe stock Android overwriting the
new recovery.

The device was booted directly into recovery, and standalone TWRP loaded
successfully.

## Disaster-Recovery Backup

Status: **CONFIRMED**

Before replacing the operating system, standalone TWRP created a private backup
on removable storage. The retained categories include:

* Boot
* EFS
* Modem and modem-state data
* Firmware data
* Associated recovery metadata

The backup was also copied to the Linux host and the transfer completed without
skipped files.

These artifacts are device-specific disaster-recovery assets. Raw backup files,
identity-bearing contents, unique backup paths, and private artifact hashes must
not be committed or reproduced in public documentation. Durable private archival
and an additional offline or encrypted copy remain planned.

## LineageOS 18.1

Status: **CONFIRMED**

The selected ROM was the final official signed `klte` LineageOS 18.1 build:

```text
lineage-18.1-20240306-nightly-klte-signed.zip
Android 11
SHA256: 872ba6483e1c022184108d8581a6a7d7176c46e6db0dd4422c9764382715cd3b
```

Official LineageOS maintenance for this device has ended. The installed build
is therefore a historical, discontinued release even though it remains an
officially signed LineageOS artifact.

TWRP wiped Dalvik/ART cache, system, data, and cache while preserving internal
and removable storage. The signed ZIP was installed from removable storage,
automatic reboot was disabled, and the optional TWRP app was not installed.

The first LineageOS boot reached setup successfully. Configuration remained
minimal and appliance-oriented: usage diagnostics, recovery auto-update, and
biometrics were declined. The installed system subsequently reported the
`kltevzw` device property.

## De-Googled Application Policy

Status: **CONFIRMED DESIGN DECISION**

Google Apps were deliberately not installed. The working policy is:

* Start with pure LineageOS.
* Prefer F-Droid and official direct APK distribution.
* Consider Aurora only if it adds practical value.
* Consider microG only if a required application needs Play Services APIs.
* Avoid disturbing a stable appliance state for speculative compatibility.

This reduces background load and telemetry on old hardware while preserving the
small application set needed for the endpoint role. LineageOS 18.1 is the
current working ROM; this record does not claim it is universally the best
long-term ROM.

## ADB and Linux udev Integration

Status: **CONFIRMED**

After LineageOS installation, ADB presented with USB ID `18d1:4ee7` under
Google's USB vendor ID rather than the earlier Samsung `04e8` presentation. An
additional udev rule was needed for host permissions.

A public-safe representation of the rule is:

```udev
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="plugdev", TAG+="uaccess"
```

After applying the rule and accepting the device authorization prompt, ADB
reported the device as authorized. The ADB serial is intentionally omitted.

### Fish Remote-Wildcard Rule

Status: **CONFIRMED**

Wildcards intended for the Android shell must be quoted so fish does not expand
them on the Linux host.

Incorrect:

```fish
adb shell chmod 755 /data/local/tmp/*
```

Correct:

```fish
adb shell 'chmod 755 /data/local/tmp/*'
```

This is a reusable host-integration lesson for future Android endpoint work.

## Magisk Root

Status: **CONFIRMED**

The original `boot.img` was extracted from the exact installed LineageOS ZIP
and patched on-device with Magisk 30.7. Heimdall then flashed the patched image
to the boot partition with `--no-reboot`.

Magisk initially denied ADB-shell elevation because an earlier superuser prompt
had been missed and remembered as denied. After resetting and explicitly
granting that permission, the sanitized validation command was:

```fish
adb shell su -c id
```

It returned a root UID and the expected Magisk SELinux context. Private patched
boot images and device-specific artifact metadata remain outside the public
repository.

## F-Droid and Moonlight Integration

Status: **CONFIRMED**

F-Droid was installed from its official APK, and its initial repository refresh
completed successfully. Moonlight Game Streaming was then installed through
F-Droid.

Moonlight immediately discovered the `basement-node` Sunshine host over the
LAN, and pairing succeeded. This validates the primary endpoint concept for the
recorded project period:

* Wi-Fi and local discovery operated sufficiently for host discovery.
* Moonlight operated on the installed LineageOS build.
* The Android client interoperated with the Sunshine host.
* The Galaxy S5 could perform its intended baseline thin-client role.

This result does not prove current network conditions or long-term streaming
performance.

## Performance Observation

Status: **OBSERVED**

The device felt substantially more responsive after replacing the stock
Samsung/Verizon system with the lean LineageOS configuration.

The working interpretation is that reduced carrier and background-service
overhead materially improved usability. This is a qualitative observation, not
a benchmark or a claim that 2014 hardware performs like a modern device.

## Appliance Integration Backlog

### MHL and External Display

Status: **PLANNED / BACKLOG**

The SM-G900V hardware is documented as supporting Samsung's 11-pin MHL
arrangement. Generic 5-pin MHL and app-based USB display adapters are not
equivalent.

Whether the installed LineageOS 18.1 configuration preserves working MHL output
has not been tested. This document claims neither success nor failure.

A stock Verizon Galaxy S3 that also supports Samsung 11-pin MHL is available as
a control device. The planned controlled test is:

1. Test the stock control device with one adapter, HDMI cable, power source, and
   display input.
2. Test the LineageOS Galaxy S5 with the same equipment.
3. If the control succeeds and the S5 fails, investigate the installed ROM and
   device support before drawing a conclusion.

### Bluetooth Input

Status: **PLANNED / BACKLOG**

Bluetooth keyboard and pointing-device behavior has not been validated. A
permanent peripheral should be Android-compatible, couch-friendly, reliable on
wake/reconnect, and avoid consuming the phone's port with a proprietary receiver.

### Tailscale

Status: **PLANNED / BACKLOG**

Tailscale has not been fully installed or authenticated on this device. The
preferred future route is the official upstream universal Android APK, with
Android 11 compatibility verified before installation.

Tailnet identifiers, authentication keys, and device-authentication material
must not appear in this repository.

### Terminal and SSH Client

Status: **PLANNED / BACKLOG**

A terminal or SSH client remains desirable for keyboard-assisted administration.
No client has been selected. Proprietary and open-source options should be
evaluated from official distribution sources rather than third-party APK
mirrors.

### Browser-Based Assistant Access

Status: **PLANNED / BACKLOG**

For this deliberately de-Googled device, a browser or home-screen shortcut is
preferred over installing a native application from an untrusted APK mirror.
Play-services compatibility should be revisited only if a required application
justifies it.

### Power, Battery, and Mounting

Status: **PLANNED / BACKLOG**

Open work includes battery replacement, safe continuous-power behavior,
charge-limiting options, airflow, thermal placement, screen timeout/wake
behavior, and a reversible mounting strategy.

An aged lithium battery must not be trapped against a warm display or inside an
unventilated enclosure.

### Moonlight Tuning

Status: **PLANNED / BACKLOG**

The initial tuning target is 1080p60, H.264, and a moderate bitrate. Future
tests should record latency, frame drops, decode time, Wi-Fi stability, input
behavior, and external-display behavior if MHL is validated.

There is no operational reason to target 4K on this endpoint unless controlled
testing demonstrates a useful result.

### Durable Private Artifact Archival

Status: **PLANNED / BACKLOG**

Firmware, recovery, ROM, root-tool, and disaster-backup artifacts need durable
private archival outside transient working locations and outside Git.

Before moving them:

1. Inspect the intended private destination without exposing its contents.
2. Copy rather than delete the working artifacts.
3. Validate hashes at the destination.
4. Retain the working copies until the archival result is confirmed.
5. Consider an additional offline or encrypted recovery copy.

Private binaries, device-specific backups, and identity-bearing metadata must
never be committed to this public repository.

## Engineering Lessons

### Identify the exact variant and CID path first

The CID15 finding determined whether the historical bootloader-unlock route was
applicable. Generic Galaxy S5 guides were not an adequate authority for the
Verizon variant.

### Preserve recovery options before destructive changes

Live PIT inspection, fallback firmware, and a private backup of boot, EFS,
modem, modem-state, and firmware data reduced the recovery risk.

### Inspect historical tooling

Static inspection exposed obsolete network dependencies, clarified the root and
unlock sequence, and prevented blind execution of duplicate low-level operations.

### Validate from device state

Download Mode indicators and direct root checks established the outcome. Script
completion messages and static analysis were supporting evidence, not the sole
authority.

### Control boot sequencing

Heimdall `--no-reboot` kept sensitive recovery and boot transitions deliberate.
The first boot after flashing standalone recovery went directly to TWRP as a
precaution against known Samsung stock-recovery restoration behavior.

### Match Magisk to the installed ROM

The patched boot image came from the exact LineageOS ZIP installed on the
device, avoiding a cross-build boot-image mismatch.

### Keep the appliance minimal

The confirmed F-Droid and Moonlight path showed that GApps were not required for
the primary thin-client role.

## Current Project Status

```text
BOOTLOADER: COMPLETE
CUSTOM RECOVERY: COMPLETE
PRIVATE DISASTER BACKUP: COMPLETE
LINEAGEOS: COMPLETE
DE-GOOGLE: COMPLETE
MAGISK ROOT: COMPLETE
ADB: COMPLETE
F-DROID: COMPLETE
MOONLIGHT: FUNCTIONAL
SUNSHINE DISCOVERY: FUNCTIONAL
MHL: PLANNED / BACKLOG
BLUETOOTH INPUT: PLANNED / BACKLOG
TAILSCALE: PLANNED / BACKLOG
SSH CLIENT: PLANNED / BACKLOG
PERMANENT INSTALLATION: PLANNED / BACKLOG
DURABLE PRIVATE ARCHIVAL: PLANNED / BACKLOG
```

## Public Documentation Checklist

Before publishing future updates, confirm that the document does not contain:

* IMEI or serial-number values
* Full raw CID or ADB serial values
* Raw EFS, modem, or other identity-bearing data
* Credentials, Wi-Fi secrets, private keys, or authentication material
* Private network topology or unnecessary host inventory
* Private recovery binaries, backup archives, or unique backup paths
* Raw private logs or personal media inventories

## Final Assessment

The Verizon Galaxy S5 SM-G900V was transformed from a carrier-controlled stock
Android 6-era phone into a bootloader-unlocked, standalone-recovery-capable,
privately disaster-backed-up, Android 11, de-Googled, Magisk-rooted,
ADB-manageable, Moonlight-capable endpoint.

The device can now be treated as an Android homelab endpoint rather than merely
as an old phone. The platform-conversion phase is complete. External display,
Bluetooth input, Tailscale, SSH tooling, permanent power and mounting,
Moonlight tuning, and durable private archival remain future work.

## Related Documentation

* [Endpoint and Client Devices](../README.md)
* [Architecture Overview](../../architecture-overview.md)
* [Access and Security](../../access-and-security.md)
* [Interactive Access Observations](../../interactive-access-observations.md)
