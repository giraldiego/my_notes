```markdown
---
title: Sync Dropbox with Google Drive using rclone bisync
date: 2025-01-24
tags:
  - linux
  - fedora
  - sync
  - dropbox
  - google-drive
  - rclone
  - systemd
---

# Sync Dropbox Folder with Google Drive using rclone bisync

> [!info] Overview
> This guide sets up **two-way sync** between a local Dropbox folder and a Google Drive cloud folder using `rclone bisync` on **Fedora 43**.
>
> Changes from either side (local edits, Google Drive web, shared users) are synced automatically every 5 minutes.

```
~/Dropbox/MyFolder ←→ Dropbox Cloud    (handled by Dropbox client)
~/Dropbox/MyFolder ←→ Google Drive Cloud (handled by rclone bisync)
```

> [!warning] GNOME Online Accounts
> GNOME Online Accounts streams files via GVFS and doesn't keep real local copies. `inotify` doesn't work on it. This guide skips it entirely and syncs directly with Google Drive cloud via rclone.

---

## 1. Install rclone

```bash
sudo dnf install rclone
```

---

## 2. Configure Google Drive Remote

```bash
rclone config
```

Follow the interactive prompts:

| Prompt                  | Value                        |
| ----------------------- | ---------------------------- |
| Action                  | `n` (New remote)             |
| Name                    | `gdrive`                     |
| Storage                 | `drive`                      |
| client_id               | *(press Enter for default)*  |
| client_secret           | *(press Enter for default)*  |
| scope                   | `1` (full access)            |
| root_folder_id          | *(press Enter)*              |
| service_account_file    | *(press Enter)*              |
| Edit advanced config    | `n`                          |
| Auto config             | `y`                          |

A browser window will open. **Log in to your Google account** and authorize rclone.

### Verify it works

```bash
# List your Google Drive root
rclone ls gdrive:

# List a specific folder
rclone ls gdrive:MyFolder
```

---

## 3. First Sync (Do This Carefully)

> [!caution]
> `--resync` is **required the first time only**. It establishes the baseline for future syncs.

```bash
# Dry run first — see what would happen without making changes
rclone bisync ~/Dropbox/MyFolder gdrive:MyFolder --dry-run --verbose

# If it looks good, run the initial resync
rclone bisync ~/Dropbox/MyFolder gdrive:MyFolder --resync --verbose
```

### Test a regular sync

```bash
rclone bisync ~/Dropbox/MyFolder gdrive:MyFolder --verbose
```

---

## 4. Set Up the Safety Check File

This prevents rclone from wiping everything if one side appears empty (e.g., during a network glitch):

```bash
touch ~/Dropbox/MyFolder/RCLONE_TEST
rclone copy ~/Dropbox/MyFolder/RCLONE_TEST gdrive:MyFolder/
```

> [!important]
> Do **not** delete the `RCLONE_TEST` file from either side. It's used by `--check-access` to verify both sides are reachable.

---

## 5. Create the Sync Script

```bash
mkdir -p ~/bin
mkdir -p ~/.local/log
nano ~/bin/bisync-dropbox-gdrive.sh
```

Paste the following:

```bash
#!/bin/bash

LOCAL="$HOME/Dropbox/MyFolder"
REMOTE="gdrive:MyFolder"
LOG="$HOME/.local/log/bisync.log"
LOCKFILE="/tmp/bisync.lock"

mkdir -p "$(dirname "$LOG")"

# Prevent overlapping runs
if [ -f "$LOCKFILE" ]; then
    echo "[$(date)] Sync already running, skipping." >> "$LOG"
    exit 0
fi
trap 'rm -f "$LOCKFILE"' EXIT
touch "$LOCKFILE"

echo "[$(date)] Starting bisync..." >> "$LOG"

rclone bisync "$LOCAL" "$REMOTE" \
    --verbose \
    --check-access \
    --conflict-resolve newer \
    --conflict-loser num \
    --conflict-suffix sync-conflict \
    --resilient \
    --recover \
    --max-lock 2m \
    2>&1 | tee -a "$LOG"

echo "[$(date)] Sync complete. Exit code: $?" >> "$LOG"
```

Make it executable:

```bash
chmod +x ~/bin/bisync-dropbox-gdrive.sh
```

### Key Flags

| Flag                          | Purpose                                            |
| ----------------------------- | -------------------------------------------------- |
| `--check-access`              | Safety check to prevent accidental mass deletions  |
| `--conflict-resolve newer`    | Newer file wins on conflicts                       |
| `--conflict-loser num`        | Keeps the losing file with a numbered suffix       |
| `--conflict-suffix sync-conflict` | Names conflict copies clearly                 |
| `--resilient`                 | Handles transient errors gracefully                |
| `--recover`                   | Auto-recovers from interrupted syncs               |
| `--max-lock 2m`               | Times out stale locks after 2 minutes              |

---

## 6. Create systemd Service and Timer

### Service

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/bisync.service
```

```ini
[Unit]
Description=Bisync Dropbox with Google Drive
After=network-online.target

[Service]
Type=oneshot
ExecStart=%h/bin/bisync-dropbox-gdrive.sh
Nice=10
CPUQuota=30%
```

### Timer

```bash
nano ~/.config/systemd/user/bisync.timer
```

```ini
[Unit]
Description=Run bisync every 5 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```

### Enable and Start

```bash
systemctl --user daemon-reload
systemctl --user enable --now bisync.timer
```

---

## 7. Monitoring

```bash
# Check timer status
systemctl --user list-timers

# Check last sync result
systemctl --user status bisync.service

# Watch logs live
tail -f ~/.local/log/bisync.log

# Journal
journalctl --user -u bisync -f
```

---

## 8. Disable GNOME Online Accounts for Google Drive (Optional)

Since the sync is handled by rclone now:

1. Go to **Settings → Online Accounts → Google**
2. Toggle **off** "Files"
3. Keep Calendar/Contacts toggled on if you use them

---

## Sync Behavior Reference

| Scenario                                 | Result                                                     |
| ---------------------------------------- | ---------------------------------------------------------- |
| You edit a file in `~/Dropbox/MyFolder`  | → Synced to Google Drive ✅                                |
| Someone edits a file via Google Drive web | → Pulled to `~/Dropbox/MyFolder` ✅                       |
| A shared user adds a file in Google Drive | → Appears in Dropbox ✅                                   |
| You delete a file locally                | → Deleted from Google Drive ✅                             |
| **Conflict**: both sides edited          | → Newer wins, older saved as `*.sync-conflict` ✅         |
| Network is down                          | → Sync retries next cycle ✅                               |
| Google Drive appears empty (glitch)      | → `--check-access` **blocks the sync** to protect files ✅ |

---

## Adjusting the Sync Interval

Edit the timer file:

```bash
nano ~/.config/systemd/user/bisync.timer
```

Change `OnUnitActiveSec` to your preference:

```ini
# Every 2 minutes (more responsive)
OnUnitActiveSec=2min

# Every 5 minutes (default, balanced)
OnUnitActiveSec=5min

# Every 10 minutes (lighter on API usage)
OnUnitActiveSec=10min
```

Reload after editing:

```bash
systemctl --user daemon-reload
systemctl --user restart bisync.timer
```

> [!note]
> Google Drive API has a quota of ~10,000 requests per day. Every 5 minutes is perfectly safe for normal-sized folders.

---

## Troubleshooting

### Sync fails with "needs --resync"

Something went wrong with the baseline. Re-run:

```bash
rclone bisync ~/Dropbox/MyFolder gdrive:MyFolder --resync --verbose
```

### Stale lock file

```bash
rm -f /tmp/bisync.lock
```

### Check rclone config

```bash
rclone config show
rclone listremotes
```

### Test connectivity

```bash
rclone about gdrive:
```

### Log rotation (prevent huge log files)

Add to your crontab:

```bash
crontab -e
```

```cron
0 0 * * 0 truncate -s 1M ~/.local/log/bisync.log
```

This truncates the log to 1MB every Sunday at midnight.
```