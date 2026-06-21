# MiniDLNA Indexing Retry Automation

## Purpose

This document records the MiniDLNA indexing reliability work completed on `basement-node`.

The goal was to improve MiniDLNA behavior when video files existed on disk but were not present in the MiniDLNA database. The final design avoids repeated MiniDLNA restarts, avoids frequent full database rebuilds, and uses DB-aware retry automation to trigger indexing only for files missing from MiniDLNA’s SQLite database.

## Summary

Completed outcomes:

* Verified media mounts and MiniDLNA service state.
* Confirmed MiniDLNA was active and configured for video-only media directories.
* Confirmed the expected files existed on disk but were not all visible through the DLNA client.
* Inspected the existing user-level MiniDLNA nudge timer.
* Identified that the existing timer only handled recently modified files and used timestamp-only `touch` events.
* Tested MiniDLNA filesystem event behavior.
* Confirmed MiniDLNA did not reliably retry indexing from timestamp-only `touch` changes.
* Confirmed rename events triggered MiniDLNA indexing.
* Inspected MiniDLNA’s SQLite database at `/var/cache/minidlna/files.db`.
* Created a DB-aware retry script at `/usr/local/bin/minidlna-db-nudge`.
* Created a fast system-level nudge timer for active intake folders.
* Created a daily recursive audit timer for all configured media roots.
* Disabled and removed the old user-level nudge timer from the active user systemd path.
* Validated MiniDLNA and both new timers were active.
* Created a Timeshift snapshot after the automation changes.

## System Context

* Hostname: `basement-node`
* User: `sona`
* OS: Pop!_OS 24.04 LTS
* Shell: fish
* Service: MiniDLNA / ReadyMedia
* MiniDLNA database path: `/var/cache/minidlna/files.db`
* Shared retry script: `/usr/local/bin/minidlna-db-nudge`

Configured MiniDLNA media paths:

```text
media_dir=V,/mnt/media-primary/Downloads
media_dir=V,/mnt/media-primary/VRVideos
media_dir=V,/mnt/media-secondary/Movies
inotify=yes
```

Media mount points:

```text
/mnt/media-primary
/mnt/media-secondary
```

## Architecture / Service Model

MiniDLNA uses configured media directories and maintains an SQLite database at:

```text
/var/cache/minidlna/files.db
```

The retry automation compares video files on disk against MiniDLNA’s database. If a file exists on disk but is missing from the database, the automation performs a rename-and-rename-back operation to generate a filesystem event MiniDLNA responds to.

Final service model:

```text
Configured media folders
→ /usr/local/bin/minidlna-db-nudge
→ compare filesystem paths against /var/cache/minidlna/files.db
→ rename only files missing from MiniDLNA DB
→ MiniDLNA inotify detects rename event
→ MiniDLNA indexes missing file
```

Two timers are used:

```text
minidlna-fast-nudge.timer
→ runs every 2 minutes
→ top-level scan only
→ intended for active intake folders

minidlna-audit-nudge.timer
→ runs daily at 03:30
→ recursive scan
→ intended to catch sorted or previously missed files
```

The design avoids:

* Frequent MiniDLNA restarts.
* Frequent full database rebuilds.
* Repeated renaming of files already present in the MiniDLNA database.
* Frequent recursive scans of the full media tree.

## Starting State

MiniDLNA was installed, enabled, and active.

Service check:

```fish
systemctl status minidlna --no-pager
```

Observed result:

```text
minidlna.service loaded/enabled
Active: active (running)
HTTP listening on port 8200
```

Media mount verification:

```fish
findmnt /mnt/media-primary
findmnt /mnt/media-secondary
```

Observed result:

```text
/mnt/media-primary   /dev/sda2   ext4   rw,relatime
/mnt/media-secondary /dev/sdb2   ext4   rw,relatime
```

MiniDLNA config verification:

```fish
grep -E '^(media_dir|inotify|db_dir|log_dir)' /etc/minidlna.conf
```

Observed result:

```text
media_dir=V,/mnt/media-primary/Downloads
media_dir=V,/mnt/media-primary/VRVideos
media_dir=V,/mnt/media-secondary/Movies
inotify=yes
```

An existing user-level timer was present:

```fish
systemctl --user list-timers | grep -i nudge
```

Observed result showed:

```text
nudge-minidlna-downloads.timer
nudge-minidlna-downloads.service
```

## Initial File Count Validation

The expected files existed on disk.

Commands:

```fish
echo "Downloads video count:"
find /mnt/media-primary/Downloads -maxdepth 1 -type f | wc -l

echo "VRVideos video count:"
find /mnt/media-primary/VRVideos -maxdepth 1 -type f | wc -l

echo "Movies video count:"
find /mnt/media-secondary/Movies -maxdepth 1 -type f | wc -l
```

Observed result:

```text
Downloads video count:
23

VRVideos video count:
0

Movies video count:
0
```

The client view showed only a subset of the files. This confirmed the issue was not missing files on disk.

## Existing User Timer Inspection

Inspected the existing user timer, service, and script:

```fish
systemctl --user cat nudge-minidlna-downloads.timer
systemctl --user cat nudge-minidlna-downloads.service
sed -n '1,220p' /home/sona/bin/nudge-minidlna-downloads
```

Timer content:

```ini
[Unit]
Description=Run MiniDLNA download nudge every 2 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=2min
AccuracySec=30s
Persistent=true

[Install]
WantedBy=timers.target
```

Service content:

```ini
[Unit]
Description=Nudge completed video downloads for MiniDLNA indexing

[Service]
Type=oneshot
ExecStart=/home/sona/bin/nudge-minidlna-downloads
```

Existing script behavior:

* Scanned `/mnt/media-primary/Downloads`.
* Scanned `/mnt/media-primary/VRVideos`.
* Scanned `/mnt/media-secondary/Movies`.
* Included subdirectories.
* Only touched files modified within the last 180 minutes.
* Skipped partial/temp files.
* Skipped files open in `lsof`.
* Ran `chmod 664`.
* Checked readability as `minidlna`.
* Used `touch` to update file modification time.

Issue identified:

```text
The existing script only retried recently modified files and used timestamp-only touch events. Older files missed by MiniDLNA would not be retried, and touch events were later confirmed insufficient for reliable MiniDLNA indexing retries.
```

## Filesystem Event Testing

A timestamp-only `touch` test did not cause the missing files to appear through MiniDLNA.

Operational result:

```text
MiniDLNA did not reliably treat timestamp-only mtime changes as sufficient to retry indexing missed files.
```

A rename test was then performed on a missing file in the watched directory.

Operational result:

```text
Renaming a file triggered MiniDLNA indexing.
Touching a file did not.
```

Conclusion:

```text
MiniDLNA responded to rename-style filesystem events but did not reliably respond to timestamp-only touch changes for missed files.
```

## MiniDLNA SQLite Database Inspection

`sqlite3` was used to inspect the MiniDLNA database.

Database path:

```text
/var/cache/minidlna/files.db
```

Commands:

```fish
sudo sqlite3 /var/cache/minidlna/files.db ".tables"
sudo sqlite3 /var/cache/minidlna/files.db ".schema DETAILS"
```

Tables found:

```text
ALBUM_ART
BOOKMARKS
CAPTIONS
DETAILS
OBJECTS
PLAYLISTS
SETTINGS
```

Relevant schema:

```sql
CREATE TABLE DETAILS (
  ID INTEGER PRIMARY KEY AUTOINCREMENT,
  PATH TEXT DEFAULT NULL,
  SIZE INTEGER,
  TIMESTAMP INTEGER,
  TITLE TEXT COLLATE NOCASE,
  DURATION TEXT,
  BITRATE INTEGER,
  SAMPLERATE INTEGER,
  CREATOR TEXT COLLATE NOCASE,
  ARTIST TEXT COLLATE NOCASE,
  ALBUM TEXT COLLATE NOCASE,
  GENRE TEXT COLLATE NOCASE,
  COMMENT TEXT,
  CHANNELS INTEGER,
  DISC INTEGER,
  TRACK INTEGER,
  DATE DATE,
  RESOLUTION TEXT,
  THUMBNAIL BOOL DEFAULT 0,
  ALBUM_ART INTEGER DEFAULT 0,
  ROTATION INTEGER,
  DLNA_PN TEXT,
  MIME TEXT
);

CREATE INDEX IDX_DETAILS_PATH ON DETAILS(PATH);
CREATE INDEX IDX_DETAILS_ID ON DETAILS(ID);
```

Confirmed:

```text
DETAILS table exists.
PATH column exists.
PATH index exists.
MiniDLNA indexed file paths can be queried directly.
```

## Changes Completed

### Created Shared DB-Aware Retry Script

Created:

```text
/usr/local/bin/minidlna-db-nudge
```

Script purpose:

```text
Compare filesystem video files against /var/cache/minidlna/files.db.
If a file is missing from MiniDLNA DB, rename it to create a real filesystem event.
Rename it back immediately.
Skip recent files.
Skip open files.
Skip unreadable files.
Support fast and audit modes.
```

Supported modes:

```text
--mode fast
--mode audit
```

Mode behavior:

```text
fast  → top-level scan only
audit → recursive scan
```

Made executable:

```fish
sudo chmod +x /usr/local/bin/minidlna-db-nudge
```

Manual fast test:

```fish
sudo /usr/local/bin/minidlna-db-nudge --mode fast
```

Manual audit test:

```fish
sudo /usr/local/bin/minidlna-db-nudge --mode audit
```

Audit result:

```text
Checked video files: 1631
Missing from MiniDLNA DB: 1
```

This confirmed the script did not brute-force rename the full library and acted only on files missing from the MiniDLNA database.

### Created Fast Nudge Service and Timer

Created service:

```text
/etc/systemd/system/minidlna-fast-nudge.service
```

Purpose:

```text
Run DB-aware top-level MiniDLNA nudge.
```

Created timer:

```text
/etc/systemd/system/minidlna-fast-nudge.timer
```

Timer behavior:

```text
Runs every 2 minutes.
```

### Created Recursive Audit Service and Timer

Created service:

```text
/etc/systemd/system/minidlna-audit-nudge.service
```

Purpose:

```text
Run DB-aware recursive MiniDLNA audit.
```

Created timer:

```text
/etc/systemd/system/minidlna-audit-nudge.timer
```

Final audit timer configuration:

```ini
[Unit]
Description=Run MiniDLNA recursive audit nudge daily

[Timer]
OnCalendar=*-*-* 03:30:00
AccuracySec=10min
Persistent=true
Unit=minidlna-audit-nudge.service

[Install]
WantedBy=timers.target
```

Enabled timers:

```fish
sudo systemctl daemon-reload
sudo systemctl enable --now minidlna-fast-nudge.timer
sudo systemctl enable --now minidlna-audit-nudge.timer
```

## Timer Validation

Commands:

```fish
systemctl status minidlna-fast-nudge.timer --no-pager
systemctl status minidlna-audit-nudge.timer --no-pager
systemctl list-timers | grep -i minidlna
```

Observed result:

```text
minidlna-fast-nudge.timer active/waiting
Next run: every 2 minutes

minidlna-audit-nudge.timer active/waiting
Next run: Mon 2026-06-22 03:30:00 EDT
```

Example timer output:

```text
Sun 2026-06-21 12:59:37 EDT ... minidlna-fast-nudge.timer  minidlna-fast-nudge.service
Mon 2026-06-22 03:30:00 EDT ... minidlna-audit-nudge.timer minidlna-audit-nudge.service
```

## Old User Timer Cleanup

Verified old user timer state:

```fish
systemctl --user is-enabled nudge-minidlna-downloads.timer
systemctl --user is-active nudge-minidlna-downloads.timer
systemctl --user status nudge-minidlna-downloads.timer --no-pager
```

Observed result:

```text
disabled
inactive
```

Checked for old timer symlink:

```fish
ls -l /home/sona/.config/systemd/user/timers.target.wants/ | grep -i nudge
```

Observed result:

```text
No active timer symlink found.
```

Moved old user units out of the active path:

```fish
mkdir -p /home/sona/systemd-user-disabled-backups

mv /home/sona/.config/systemd/user/nudge-minidlna-downloads.timer /home/sona/systemd-user-disabled-backups/
mv /home/sona/.config/systemd/user/nudge-minidlna-downloads.service /home/sona/systemd-user-disabled-backups/

systemctl --user daemon-reload
systemctl --user reset-failed
```

Final cleanup check:

```fish
systemctl --user list-unit-files | grep -i nudge
systemctl --user list-timers | grep -i nudge
```

Observed result:

```text
No nudge user units listed.
No nudge user timers listed.
```

Conclusion:

```text
The old user-level nudge timer is no longer active and should not return after reboot. The new system-level timers are now the active automation path.
```

## Final Service Validation

Commands:

```fish
systemctl is-active minidlna
systemctl is-active minidlna-fast-nudge.timer
systemctl is-active minidlna-audit-nudge.timer
systemctl list-timers | grep -i minidlna
```

Observed result:

```text
active
active
active
```

Confirmed timer schedule:

```text
minidlna-fast-nudge.timer  → active, every 2 minutes
minidlna-audit-nudge.timer → active, daily at 03:30 EDT
```

## Timeshift Snapshot

Created a known-good Timeshift snapshot after MiniDLNA automation was validated.

Command:

```fish
sudo timeshift --create --comments "Known good after MiniDLNA DB-aware nudge timers and VR sim racing validation" --tags B
```

Snapshot result:

```text
RSYNC Snapshot saved successfully
Tagged snapshot '2026-06-21_12-58-01': ondemand
```

Timeshift pruned an older untagged snapshot due to retention limits:

```text
Removed '2026-06-09_16-56-53'
```

Verified snapshot list:

```fish
sudo timeshift --list
```

Important remaining snapshots:

```text
2026-06-17_14-33-39  B H D  Known good after Cloudflare noVNC persistence services
2026-06-21_12-58-01  B      Known good after MiniDLNA DB-aware nudge timers and VR sim racing validation
```

Current Timeshift status:

```text
Mode: RSYNC
Status: OK
11 snapshots
678.5 GB free
```

## Validation Completed

Validated during the session:

* `/mnt/media-primary` and `/mnt/media-secondary` were mounted read/write as ext4.
* MiniDLNA was active and listening on port 8200.
* MiniDLNA config included the expected video media paths.
* MiniDLNA `inotify=yes` remained configured.
* Expected files existed on disk.
* Existing user timer was inspected and identified as insufficient.
* `touch` did not reliably trigger MiniDLNA retry behavior.
* Rename events triggered MiniDLNA indexing.
* MiniDLNA SQLite database was readable.
* `DETAILS.PATH` existed and could support DB-aware file comparison.
* `/usr/local/bin/minidlna-db-nudge` was created and made executable.
* Manual audit mode checked 1631 files and found 1 missing MiniDLNA DB entry.
* `minidlna-fast-nudge.timer` was enabled and active.
* `minidlna-audit-nudge.timer` was enabled and active.
* Old user-level nudge units were disabled, inactive, and moved out of the active user systemd path.
* No old user nudge units or timers remained listed.
* MiniDLNA and both new timers returned `active`.
* Timeshift snapshot `2026-06-21_12-58-01` was created.

## Current Working State

MiniDLNA:

```text
active
video-only media paths configured
inotify=yes
```

New automation:

```text
minidlna-fast-nudge.timer
  system-level timer
  active
  runs every 2 minutes
  top-level scan only
  DB-aware
  renames only files missing from MiniDLNA DB
```

```text
minidlna-audit-nudge.timer
  system-level timer
  active
  runs daily at 03:30 EDT
  recursive scan
  DB-aware
  renames only files missing from MiniDLNA DB
```

Old automation:

```text
nudge-minidlna-downloads.timer
  user-level timer
  disabled
  inactive
  moved to /home/sona/systemd-user-disabled-backups/
  no longer listed in user timers
```

Rollback:

```text
Timeshift snapshot:
2026-06-21_12-58-01
```

## Still Needs Verification

Concrete remaining checks:

* Verify behavior after a reboot:

  * MiniDLNA starts.
  * `minidlna-fast-nudge.timer` starts.
  * `minidlna-audit-nudge.timer` remains scheduled for 03:30.
  * Old user-level nudge timer does not return.
* Confirm the next newly added top-level video appears through MiniDLNA without manually restarting MiniDLNA.
* Confirm the next daily audit run completes successfully at 03:30.

## Operational Notes

MiniDLNA retry behavior observed during testing:

```text
timestamp-only touch event → not reliable for retry indexing
rename event               → triggered indexing
```

Useful inspection commands:

```fish
systemctl is-active minidlna
systemctl is-active minidlna-fast-nudge.timer
systemctl is-active minidlna-audit-nudge.timer
systemctl list-timers | grep -i minidlna
```

```fish
systemctl --user list-unit-files | grep -i nudge
systemctl --user list-timers | grep -i nudge
```

```fish
journalctl -u minidlna-fast-nudge.service --since "24 hours ago" --no-pager
journalctl -u minidlna-audit-nudge.service --since "24 hours ago" --no-pager
journalctl -u minidlna --since "24 hours ago" --no-pager
```

```fish
sudo sqlite3 /var/cache/minidlna/files.db ".tables"
sudo sqlite3 /var/cache/minidlna/files.db ".schema DETAILS"
```

Design notes:

* The final design avoids MiniDLNA service restarts.
* The final design avoids repeated full database rebuilds.
* SQLite comparison prevents unnecessary repeated renames for files already indexed.
* Fast and audit behavior are intentionally separated to balance responsiveness and resource usage.
* Recursive audit is daily because a manual audit checked 1631 files and only found 1 missing entry.

## Skills Practiced

* Linux service troubleshooting
* systemd user timer cleanup
* systemd system timer creation
* systemd timer scheduling
* MiniDLNA / ReadyMedia behavior validation
* SQLite database inspection
* DB-aware automation design
* Filesystem event testing
* Rollback planning with Timeshift
* Operational validation and documentation

## Summary

MiniDLNA indexing reliability was improved by replacing the older user-level timestamp-touch nudge with DB-aware system-level retry automation.

Testing showed MiniDLNA did not reliably retry missed files from timestamp-only `touch` changes, but did respond to rename events. A new shared script at `/usr/local/bin/minidlna-db-nudge` now compares filesystem video files against MiniDLNA’s SQLite database and renames only files missing from the database.

Two system-level timers now handle the automation: `minidlna-fast-nudge.timer` runs every 2 minutes for top-level intake paths, and `minidlna-audit-nudge.timer` runs daily at 03:30 for recursive auditing. The old user-level nudge timer was disabled and moved out of the active user systemd path.

MiniDLNA and both new timers were validated as active, and a known-good Timeshift snapshot was created after the automation changes.
