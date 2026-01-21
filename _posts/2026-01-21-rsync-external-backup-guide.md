---
layout: post
draft: false
title: "Rsync External Drive Backup with Automated Reminders"
date: 2026-01-21
description: "This post documents the process I went through to provide a way to manually sync my most important folders on my laptop to an external drive so that if I ever changed Linux distribution, I could easily copy them all back and they would be up to date to the last rsync I ran, which is weekly."
---


## Rsync External Drive Backup System

### Overview

This guide documents a reliable, efficient backup system that mirrors critical folders from your Linux Mint laptop to an external SSD using rsync. The system includes automated weekly reminders and comprehensive logging.

### What This System Does

- **Syncs specific folders** from your home directory to an external drive
- **Mirrors changes only** - fast and efficient (only changed files are transferred)
- **Maintains exact copies** - external drive matches internal drive exactly
- **Logs all operations** - timestamped records of every backup
- **Protects external-only data** - prevents accidental deletion of folders that exist only on the external drive
- **Provides weekly reminders** - automated notifications to run backups

### Use Case

Perfect for backing up personal files, documents, scripts, and configurations to an external drive that you can:
- Use for disaster recovery
- Transfer to a new laptop
- Access from another machine
- Store offsite for safety

---

### Prerequisites

### Required Software
- **rsync** - Usually pre-installed on Linux systems
- **notify-send** - For desktop notifications (part of libnotify-bin)
- **cron** - For scheduling reminders (pre-installed)

### Check if rsync is installed:
```bash
rsync --version
```

### Install if needed:
```bash
sudo apt update
sudo apt install rsync
```

### Hardware Requirements
- External drive (USB SSD, HDD, etc.)
- Sufficient space on external drive for your backed-up folders

---

### Initial Setup

### Step 1: Identify Your External Drive

1. **Connect your external drive**

2. **Find the mount point:**
```bash
lsblk -f
```

Look for your external drive. It will likely be mounted at something like:
- `/media/mark/Ext_T7_lpthp` (Samsung T7 SSD)
- `/media/mark/My_Passport` (WD Passport)
- `/media/username/drive-label`

3. **Note the exact path** - you'll need this for the script

### Step 2: Create the Backup Script

1. **Create a Scripts' directory if you don't have one:**
```bash
mkdir -p ~/Scripts
```

2. **Create the backup script:**
```bash
nano ~/Scripts/sync-to-external.sh
```

3. **Paste the following script:**

```bash
#!/bin/bash

# ============================
#  Backup Sync Script (Safe Version)
#  From: Your Laptop
#  To:   External Drive
# ============================

SRC_ROOT="$HOME"
DEST_ROOT="/media/mark/Ext_T7_lpthp"  # CHANGE THIS TO YOUR DRIVE PATH

# Check if external drive is mounted
if [ ! -d "$DEST_ROOT" ]; then
    echo "❌ Backup drive not found at $DEST_ROOT. Please mount it and try again."
    exit 1
fi

# Define folders to sync (source relative to $HOME)
# NOTE: Folders that exist ONLY on external drive should be commented out
#       to avoid accidental deletion via --delete
declare -A FOLDERS=(
    ["Desktop"]="Desktop"
    ["Downloads"]="Downloads"
    ["Documents"]="Documents"
    ["Scripts"]="Scripts"
    # ["timeshift"]="timeshift"                # Currently only exists on external SSD
    # ["Website-Backups"]="Website-Backups"    # Currently only exists on external SSD
)

# Create log directory
mkdir -p "$DEST_ROOT/logs"
LOGFILE="$DEST_ROOT/logs/backup_$(date +%Y%m%d_%H%M%S).log"

echo "🚀 Starting backup sync at $(date)" | tee -a "$LOGFILE"

for key in "${!FOLDERS[@]}"; do
    SRC_PATH="$SRC_ROOT/${FOLDERS[$key]}"
    DEST_PATH="$DEST_ROOT/${FOLDERS[$key]}"

    echo "📂 Syncing: $SRC_PATH -> $DEST_PATH" | tee -a "$LOGFILE"

    if [ -d "$SRC_PATH" ]; then
        mkdir -p "$DEST_PATH"
        rsync -avh --delete "$SRC_PATH/" "$DEST_PATH/" >> "$LOGFILE" 2>&1
    else
        echo "⚠️ Warning: Source folder not found: $SRC_PATH" | tee -a "$LOGFILE"
    fi
done

echo "✅ Backup completed at $(date)" | tee -a "$LOGFILE"
```

4. **Save and exit** (Ctrl+X, then Y, then Enter)

### Step 3: Customize the Script

**IMPORTANT: Edit these two sections for your setup:**

1. **Change the destination path:**
```bash
DEST_ROOT="/media/mark/Ext_T7_lpthp"  # Change to YOUR external drive path
```

2. **Modify the folders to back up:**
```bash
declare -A FOLDERS=(
    ["Desktop"]="Desktop"           # Add or remove folders as needed
    ["Downloads"]="Downloads"
    ["Documents"]="Documents"
    ["Scripts"]="Scripts"
    # Add more folders like:
    # ["Pictures"]="Pictures"
    # ["Videos"]="Videos"
    # ["Music"]="Music"
)
```

**Important Notes:**
- Only include folders that exist on your laptop
- Comment out (add `#` at start of line) any folders that exist ONLY on the external drive
- This prevents accidentally deleting external-only data

### Step 4: Make the Script Executable

```bash
chmod +x ~/Scripts/sync-to-external.sh
```

### Step 5: Test the Script

1. **Connect your external drive**

2. **Run the script manually:**
```bash
bash ~/Scripts/sync-to-external.sh
```

3. **Check the output** - you should see:
   - "🚀 Starting backup sync at..."
   - Progress for each folder
   - "✅ Backup completed at..."

4. **Verify the backup:**
```bash
ls /media/mark/Ext_T7_lpthp/
```

You should see your backed-up folders and a new `logs` directory.

5. **Check the log file:**
```bash
ls -lh /media/mark/Ext_T7_lpthp/logs/
cat /media/mark/Ext_T7_lpthp/logs/backup_*.log
```

---

### Setting Up Automated Reminders

### Why Weekly Reminders Instead of Automatic Backups?

This system uses **reminders** rather than automatic execution because:
- Ensures the external drive is connected when backup runs
- Gives you control over when backups occur
- Prevents errors when drive isn't mounted
- Allows you to verify backup completion
- You can postpone if you're in the middle of something

### Step 1: Determine Your Shell

```bash
echo $SHELL
```

**Result will be something like:**
- `/usr/bin/fish` (Fish shell - your setup)
- `/bin/bash` (Bash shell)
- `/usr/bin/zsh` (Zsh shell)

### Step 2: Open Crontab Editor

```bash
crontab -e
```

If this is your first time, it will ask you to choose an editor. Select `nano` (usually option 1).

### Step 3: Add the Reminder Entry

**For Fish Shell (your current setup):**
```cron
# Backup reminder every Wednesday at 8:00 AM
0 8 * * 3 DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'
```

**For Bash Shell:**
```cron
# Backup reminder every Wednesday at 8:00 AM
0 8 * * 3 DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus notify-send "Backup Reminder" "Time to run your external sync script."
```

### Step 4: Customize the Schedule

The cron format is: `minute hour day-of-month month day-of-week`

**Examples:**

```cron
# Every Monday at 9:00 AM
0 9 * * 1 DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'

# Every day at 6:00 PM
0 18 * * * DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'

# First day of every month at 8:00 AM
0 8 1 * * DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'

# Every Friday at 5:00 PM (end of work week)
0 17 * * 5 DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'
```

**Day of week codes:**
- 0 or 7 = Sunday
- 1 = Monday
- 2 = Tuesday
- 3 = Wednesday
- 4 = Thursday
- 5 = Friday
- 6 = Saturday

### Step 5: Save and Exit

- Press `Ctrl+X`
- Press `Y` to confirm
- Press `Enter` to save

### Step 6: Verify Cron Job

```bash
crontab -l
```

You should see your reminder entry in the output.

### Step 7: Test the Notification (Optional)

Don't wait until Wednesday! Test it now:

```bash
DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Backup Reminder" "Time to run your external sync script."'
```

You should see a desktop notification appear.

---

### Daily Workflow

### When You Get the Reminder

1. **Connect your external drive** (if not already connected)

2. **Open a terminal** (Ctrl+Alt+T)

3. **Run the backup script:**
```bash
bash ~/Scripts/sync-to-external.sh
```

4. **Watch the output** - it will show you what's being synced

5. **Wait for completion** - you'll see "✅ Backup completed at..."

6. **Optional: Check the log:**
```bash
cat /media/mark/Ext_T7_lpthp/logs/backup_$(date +%Y%m%d)*.log
```

### What Gets Backed Up?

**On every backup run, rsync:**
- ✅ Copies new files from laptop → external drive
- ✅ Updates changed files on external drive
- ✅ Deletes files from external drive that you deleted from laptop
- ✅ Skips unchanged files (very fast!)
- ✅ Preserves file permissions and timestamps

**The result:** Your external drive is an exact mirror of your selected folders.

---

### Understanding Rsync Options

### The Command Breakdown

```bash
rsync -avh --delete "$SRC_PATH/" "$DEST_PATH/"
```

**Flag explanations:**

| Flag | What It Does |
|------|--------------|
| `-a` | Archive mode - preserves permissions, timestamps, symlinks, etc. (equivalent to -rlptgoD) |
| `-v` | Verbose - shows you what's being transferred |
| `-h` | Human-readable - displays file sizes in KB, MB, GB instead of bytes |
| `--delete` | Removes files from destination that don't exist in source (makes it a true mirror) |

**Trailing slashes matter!**
- `"$SRC_PATH/"` - syncs the *contents* of the folder
- `"$SRC_PATH"` - syncs the folder itself (creates nested directory)

### Additional Useful Options

If you want to customize the rsync behavior, here are some options:

```bash
# Dry run - show what would be transferred without actually doing it
rsync -avh --delete --dry-run "$SRC_PATH/" "$DEST_PATH/"

# Exclude certain files or patterns
rsync -avh --delete --exclude='*.tmp' --exclude='.cache' "$SRC_PATH/" "$DEST_PATH/"

# Show progress during transfer
rsync -avh --delete --progress "$SRC_PATH/" "$DEST_PATH/"

# Compress data during transfer (slower but useful for network transfers)
rsync -avhz --delete "$SRC_PATH/" "$DEST_PATH/"
```

---

## Monitoring and Maintenance

### Checking Backup Logs

**View recent log files:**
```bash
ls -lht /media/mark/Ext_T7_lpthp/logs/ | head -10
```

**View a specific log:**
```bash
cat /media/mark/Ext_T7_lpthp/logs/backup_20260121_080000.log
```

**Search for warnings or errors:**
```bash
grep -i "warning\|error" /media/mark/Ext_T7_lpthp/logs/backup_*.log
```

**Check total backup size:**
```bash
du -sh /media/mark/Ext_T7_lpthp/Desktop
du -sh /media/mark/Ext_T7_lpthp/Downloads
du -sh /media/mark/Ext_T7_lpthp/Documents
du -sh /media/mark/Ext_T7_lpthp/Scripts
```

### Log Management

Logs accumulate over time. Clean up old logs periodically:

```bash
# Delete logs older than 90 days
find /media/mark/Ext_T7_lpthp/logs/ -name "backup_*.log" -mtime +90 -delete

# Or keep only the last 20 logs
cd /media/mark/Ext_T7_lpthp/logs/
ls -t backup_*.log | tail -n +21 | xargs rm -f
```

**Automate log cleanup** by adding this to the end of your backup script:

```bash
# Keep only last 30 backup logs
cd "$DEST_ROOT/logs"
ls -t backup_*.log 2>/dev/null | tail -n +31 | xargs rm -f 2>/dev/null
```

---

### Troubleshooting

### Problem: "Backup drive not found" Error

**Cause:** External drive not mounted or wrong path

**Solutions:**
1. Check if drive is connected: `lsblk -f`
2. Verify mount point: `ls /media/$USER/`
3. Update `DEST_ROOT` in script to correct path
4. Manually mount if needed: `udisksctl mount -b /dev/sdX1`

### Problem: Permission Denied Errors

**Cause:** Don't have write permissions on external drive

**Solutions:**
1. Check ownership: `ls -ld /media/mark/Ext_T7_lpthp/`
2. Fix permissions if needed:
```bash
sudo chown -R $USER:$USER /media/mark/Ext_T7_lpthp/
```

### Problem: Rsync Very Slow

**Possible causes and solutions:**

1. **USB 2.0 connection** - Use USB 3.0 or higher port
2. **Checking many unchanged files** - This is normal on first run
3. **Drive fragmentation** (unlikely on SSD) - Run `sudo fstrim -v /media/mark/Ext_T7_lpthp/` for SSDs

### Problem: Notification Doesn't Appear

**Solutions:**

1. **Test notification manually:**
```bash
DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus notify-send "Test" "This is a test"
```

2. **Check if cron job exists:**
```bash
crontab -l
```

3. **Check cron is running:**
```bash
systemctl status cron
```

4. **Check your user ID** (might not be 1000):
```bash
id -u
```
If it's different, update the cron path: `/run/user/YOUR_ID/bus`

### Problem: Accidental Data Loss

**Prevention:**
- Always do a dry run first: `rsync -avh --delete --dry-run SOURCE/ DEST/`
- Comment out folders in script that exist only on external drive
- Keep multiple backup drives (3-2-1 backup rule)

**Recovery:**
- If you have Timeshift snapshots, restore from there
- Check if files are in external drive's trash: `/media/mark/Ext_T7_lpthp/.Trash-1000/`

---

### Advanced Configurations

### Adding Email Notifications

Install mail utilities:
```bash
sudo apt install mailutils
```

Add to end of script:
```bash
# Send email summary
mail -s "Backup Completed on $(hostname)" your-email@example.com < "$LOGFILE"
```

### Creating Multiple Backup Profiles

Create different scripts for different backup scenarios:

```bash
~/Scripts/sync-to-external-work.sh      # Work-related folders
~/Scripts/sync-to-external-personal.sh  # Personal folders
~/Scripts/sync-to-external-media.sh     # Photos, videos, music
```

Each with different folder lists and schedules.

### Backing Up to Multiple Drives

Create a master script that calls the sync script for different drives:

```bash
#!/bin/bash
# backup-all-drives.sh

echo "Starting backup to all drives..."

# Backup to T7 SSD
DEST_ROOT="/media/mark/Ext_T7_lpthp" ~/Scripts/sync-to-external.sh

# Backup to second drive
DEST_ROOT="/media/mark/Backup_Drive_2" ~/Scripts/sync-to-external.sh

echo "All backups completed!"
```

### Syncing Over Network (SSH)

You can use rsync to back up to a remote server:

```bash
rsync -avh --delete -e ssh "$SRC_PATH/" user@remote-server:/backup/path/
```

Useful for backing up to a NAS or another computer on your network.

---

### Recovery Procedures

### Restoring from Backup

If you need to restore files from your external drive:

**Restore a single folder:**
```bash
rsync -avh /media/mark/Ext_T7_lpthp/Documents/ ~/Documents/
```

**Restore everything:**
```bash
rsync -avh /media/mark/Ext_T7_lpthp/Desktop/ ~/Desktop/
rsync -avh /media/mark/Ext_T7_lpthp/Downloads/ ~/Downloads/
rsync -avh /media/mark/Ext_T7_lpthp/Documents/ ~/Documents/
rsync -avh /media/mark/Ext_T7_lpthp/Scripts/ ~/Scripts/
```

**Restore to a new laptop:**
1. Connect external drive to new laptop
2. Copy backup script to new laptop
3. Run restore commands above
4. Reinstall applications as needed

### Verifying Backup Integrity

**Compare source and destination:**
```bash
rsync -avhn --delete ~/Documents/ /media/mark/Ext_T7_lpthp/Documents/
```

The `-n` flag does a dry run - it will show if there are any differences without making changes.

**Check file counts:**
```bash
find ~/Documents -type f | wc -l
find /media/mark/Ext_T7_lpthp/Documents -type f | wc -l
```

Both should return the same number.

---

### Best Practices

### ✅ Do This

- **Run backups regularly** - Weekly minimum, daily if critical data
- **Verify backups occasionally** - Check that files are actually there
- **Keep drive disconnected** - When not backing up, protect from power surges
- **Store offsite** - Keep a second backup in a different location
- **Test restoration** - Occasionally restore a file to verify process works
- **Monitor drive health** - SSDs: `sudo smartctl -a /dev/sdX`, HDDs: same command
- **Update script** - When you add important folders to your system

### ❌ Avoid This

- **Don't ignore warnings** - Check logs if you see errors
- **Don't rely on one backup** - Follow 3-2-1 rule (3 copies, 2 different media, 1 offsite)
- **Don't modify external drive manually** - Let rsync manage it
- **Don't yank the drive** - Always safely eject: `udisksctl unmount -b /dev/sdX1`
- **Don't back up to same drive as source** - Defeats the purpose

### 3-2-1 Backup Strategy

A complete backup strategy for important data:

- **3** copies of your data (original + 2 backups)
- **2** different types of storage media (internal SSD + external SSD + cloud)
- **1** copy stored offsite (cloud storage or external drive at another location)

**Example implementation:**
1. Original data on laptop SSD
2. First backup on external T7 SSD (this script)
3. Second backup on Déjà Dup to different drive or cloud service

---

### Quick Reference

### Common Commands

```bash
# Run backup
bash ~/Scripts/sync-to-external.sh

# Dry run (test without changing anything)
rsync -avh --delete --dry-run ~/Documents/ /media/mark/Ext_T7_lpthp/Documents/

# Check external drive space
df -h /media/mark/Ext_T7_lpthp

# List recent backups
ls -lht /media/mark/Ext_T7_lpthp/logs/ | head -5

# View latest log
cat /media/mark/Ext_T7_lpthp/logs/backup_$(ls -t /media/mark/Ext_T7_lpthp/logs/ | head -1)

# Check cron jobs
crontab -l

# Edit cron jobs
crontab -e

# Test notification
notify-send "Test" "This is a test notification"
```

### Script Location

```
~/Scripts/sync-to-external.sh
```

### Log Location

```
/media/mark/Ext_T7_lpthp/logs/backup_YYYYMMDD_HHMMSS.log
```

---

## Summary

You now have a robust, efficient backup system that:

- ✅ Mirrors critical folders to an external drive
- ✅ Only transfers changed files (fast and efficient)
- ✅ Maintains exact copies with deletion sync
- ✅ Logs all operations with timestamps
- ✅ Reminds you weekly to run backups
- ✅ Protects external-only data from deletion
- ✅ Provides full recovery capabilities

**Remember:** This is one part of a complete backup strategy. Consider also using:
- **Timeshift** - System snapshots
- **Déjà Dup** - Additional file backups
- **Cloud backup** - Offsite protection (Backblaze, OneDrive, etc.)

---

### Related Documentation

- [Rsync Official Documentation](https://rsync.samba.org/documentation.html)
- [Cron Tutorial](https://www.man7.org/linux/man-pages/man5/crontab.5.html)
- [Linux Backup Strategies](https://wiki.archlinux.org/title/Backup_programs)

---

*Last updated: January 21, 2026.*  
*Tested on: Linux Mint 22.2 with Fish shell*  
*External Drive: Samsung T7 Portable SSD*
