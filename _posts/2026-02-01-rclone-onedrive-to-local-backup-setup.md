---
layout: post
draft: false
title: "Using Rclone to pull down OneDrive contents to local Linux machine."
date: 2026-02-01
description: “I recently realized that I did not have a reliable, easy, and secure way to copy my OneDrive contents to my local machine.  I decided to set up Rclone for this job after reviewing options with Claude. This post shows how I did that, but be mindful that this is not a ‘generic’ setup as it is specific to my current workflows.”
---
## POST INSTALLATION AND ONGOING PROCEDURE UPDATE: 

**After you do this initially, what you want to do is run Rclone in the dry run mode to see if there's anything that needs to be updated, and then you just run the same command without the dry run flag.**

```sh
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive --dry-run -P --update --exclude "Personal Vault/**"
```
After reviewing that output, run:
```sh
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive --update --exclude "Personal Vault/**"
```


# Rclone Setup for OneDrive on Linux

## Overview

This guide documents the complete setup process for using `rclone` to access Microsoft OneDrive from Linux Mint. Rclone is a command-line tool that syncs files and directories to and from cloud storage services using their native APIs.

## Important: Customize for Your System

This guide was created for a specific setup. **You will need to adjust these paths for your own system:**

- **Username**: Replace `/home/mark/` with `/home/YOUR_USERNAME/`
- **External drive**: Replace `/media/mark/Ext_T7_lpthp/` with your own drive mount point
  - Find yours with: `lsblk` or check `/media/YOUR_USERNAME/`
- **Remote name**: "OneDrive" is used throughout - you can use any name you want
- **Drive selection**: When configuring, your OneDrive options may be numbered differently

**The core process (install → configure → authenticate → copy) is universal and works for everyone.**

## Initial Installation

```bash
sudo apt install rclone
```

## Configuration Process

### Step 1: Start Configuration

```bash
rclone config
```

You'll see:
```
No remotes found, make a new one?
n) New remote
s) Set configuration password
q) Quit config
n/s/q>
```

Choose: `n` (New remote)

### Step 2: Name Your Remote

```
name> onedrive
```

You can use any name you want - this is just the label rclone will use in commands.

### Step 3: Select Storage Type

From the numbered list of storage providers, choose:
```
Storage> 30
```

(Option 30 is "Microsoft OneDrive")

### Step 4: OAuth Client Settings

```
client_id> [press Enter - leave blank]
client_secret> [press Enter - leave blank]
```

You don't need custom OAuth credentials for personal OneDrive access.

### Step 5: Choose Region

```
region> [press Enter for default "global"]
```

Unless you're using a government or special regional OneDrive account, use the default global option.

### Step 6: Advanced Config

```
Edit advanced config?
y/n> n
```

No advanced configuration needed for basic usage.

### Step 7: Auto Configuration

```
Use auto config?
y/n> y
```

This will open a browser window for Microsoft authentication. Log in with your Microsoft account and authorize rclone.

### Step 8: Select Drive Type

```
config_type> [press Enter for default "onedrive"]
```

Choose "OneDrive Personal or Business"

### Step 9: Select Your Drive

You'll see a list of available drives:
```
config_driveid> 2
```

Choose option 2 (your main "OneDrive (personal)")

### Step 10: Confirm Configuration

```
Drive OK?
y/n> y

Keep this "onedrive" remote?
y/e/d> y
```

### Step 11: Exit Configuration

```
e/n/d/r/c/s/q> q
```

Configuration is now complete!

## Essential Workflow: The Two-Step Process

**This is the core of how you'll use rclone. Follow these two steps every time:**

### STEP 1: Dry Run (Always Do This First!)

See what would be copied WITHOUT actually copying anything:

```bash
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive --dry-run -P --exclude "Personal Vault/**"
```

This shows you:
- What files would be copied
- How much data would transfer
- If there are any errors

**Look at the output carefully before proceeding!**

### STEP 2: Actual Copy (Only After Dry Run Looks Good)

Once you're satisfied with what the dry run showed, run the real copy:

```bash
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive -P --exclude "Personal Vault/**"
```

**What each flag means:**
- `onedrive:/` - source (your OneDrive root)
- `/media/mark/Ext_T7_lpthp/OneDrive` - destination on external drive
- `--dry-run` - (dry run only) don't actually copy, just show what would happen
- `-P` - show progress during transfer
- `--exclude "Personal Vault/**"` - skip Personal Vault folder (prevents errors)

### About Personal Vault

Personal Vault requires special authentication that rclone can't handle. We exclude it to avoid error messages cluttering the output. If you need those files, download them manually from OneDrive.live.com.

## How Rclone Works Going Forward

### Key Concept: Rclone Does NOT Auto-Sync

**Important:** Rclone is NOT like Dropbox or OneDrive's native clients. It does not automatically sync in the background. You must manually run rclone commands whenever you want to sync.

### Copy vs Sync Commands

**`rclone copy`** (what we used)
- Copies files from source to destination
- Does NOT delete files at destination that don't exist at source
- Safe for one-way transfers
- Use when: You want to pull down files without risking deletion

**`rclone sync`**
- Makes destination exactly match source
- WILL DELETE files at destination that don't exist at source
- Use with caution!
- Use when: You want exact mirror of OneDrive

### Example: Update Your Local Copy

To pull down any new or changed files from OneDrive:

```bash
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive -P --exclude "Personal Vault/**"
```

### Example: Upload Local Changes to OneDrive

```bash
rclone copy /media/mark/Ext_T7_lpthp/OneDrive onedrive:/ -P
```

Note: source and destination are reversed.

### Example: Two-Way Sync (Advanced)

For true two-way sync where changes go both directions:

```bash
rclone sync /media/mark/Ext_T7_lpthp/OneDrive onedrive:/ -P --interactive
```

The `--interactive` flag will ask before making destructive changes.

## Useful Rclone Features

### Check What's Different

See what would be copied without actually copying:

```bash
rclone check /media/mark/Ext_T7_lpthp/OneDrive onedrive:/
```

### List Remote Files

Browse your OneDrive without downloading:

```bash
rclone ls onedrive:/
```

For a tree-style view:

```bash
rclone tree onedrive:/
```

List only directories:

```bash
rclone lsd onedrive:/
```

### Download Specific Folder

Instead of everything, download just one folder:

```bash
rclone copy onedrive:/Documents /media/mark/Ext_T7_lpthp/OneDrive/Documents -P
```

### Bandwidth Limiting

Limit upload/download speed if you don't want rclone hogging bandwidth:

```bash
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive -P --bwlimit 5M
```

This limits to 5 MB/s.

### Show Transfer Stats

Get detailed statistics after transfer:

```bash
rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive -P --stats-one-line
```

### Mount OneDrive as File system (Advanced)

You can mount OneDrive like a regular directory:

```bash
mkdir ~/OneDrive
rclone mount onedrive:/ ~/OneDrive --daemon
```

Files are downloaded on-demand when you access them. To unmount:

```bash
fusermount -u ~/OneDrive
```

**Warning:** Mounting can be slow over the internet and has some quirks. Best for occasional access, not primary workflow.

## Ongoing Usage Strategy

### Recommended Workflow

Since rclone doesn't auto-sync, here's a practical approach:

1. **For backups:** Run `rclone copy` manually whenever you want to back up OneDrive locally
2. **Schedule it (optional):** Set up a cron job or systemd timer if you want automatic backups
3. **Before major changes:** Always do a dry run first with `--dry-run`
4. **Keep it simple:** Use `copy` instead of `sync` unless you're certain you want exact mirroring

### Setting Up Automatic Backups (Optional)

If you want daily backups from OneDrive to your external drive:

1. Create a script: `/home/mark/scripts/backup-onedrive.sh`
```bash
#!/bin/bash
# Check if external drive is mounted
if [ -d "/media/mark/Ext_T7_lpthp" ]; then
    rclone copy onedrive:/ /media/mark/Ext_T7_lpthp/OneDrive \
      --log-file=/home/mark/logs/rclone-backup.log \
      --exclude "Personal Vault/**"
else
    echo "External drive not mounted" >> /home/mark/logs/rclone-backup.log
fi
```

2. Make it executable:
```bash
chmod +x /home/mark/scripts/backup-onedrive.sh
```

3. Add to crontab for daily 2 AM backups:
```bash
crontab -e
```

Add line:
```
0 2 * * * /home/mark/scripts/backup-onedrive.sh
```

**Note:** Only useful if your external drive stays connected.

## Why Rclone Over Other Methods

### vs rsync
- **rsync:** Only works with filesystems you can already access (local, SSH, etc.)
- **rclone:** Speaks cloud provider APIs directly (OneDrive, Google Drive, S3, etc.)
- You cannot use rsync with OneDrive unless you first mount it as a filesystem

### vs abraunegg/onedrive
- **abraunegg:** Full bidirectional sync daemon (like native OneDrive client)
- **rclone:** Simple, command-line driven, no daemon required
- For one-time or manual syncs, rclone is simpler
- For always-on sync like Dropbox, abraunegg is better

### vs Web Interface
- **Web:** Manual downloads only, no automation
- **rclone:** Scriptable, can handle large transfers, resume capability

## Common Commands Reference

```bash
# List configured remotes
rclone listremotes

# Get info about OneDrive quota/usage
rclone about onedrive:

# Download everything (excluding Personal Vault)
rclone copy onedrive:/ /path/to/local -P --exclude "Personal Vault/**"

# Upload everything
rclone copy /path/to/local onedrive:/ -P

# Sync local to match OneDrive exactly (DELETES local files not in OneDrive)
rclone sync onedrive:/ /path/to/local -P --interactive --exclude "Personal Vault/**"

# Check differences without copying
rclone check /path/to/local onedrive:/

# Browse remote files
rclone ls onedrive:/
rclone tree onedrive:/Documents

# Download specific folder
rclone copy onedrive:/Photos /path/to/local/Photos -P

# Delete old remote configuration
rclone config delete onedrive

# Re-authorize (if token expires)
rclone config reconnect onedrive:
```

## Troubleshooting

### Token Expired

If you get authentication errors after months of not using rclone:

```bash
rclone config reconnect onedrive:
```

This will refresh your OAuth token.

### Can't Access Personal Vault

This is expected. Personal Vault requires special authentication. Either:
- Download those files manually from onedrive.live.com
- Use `--exclude "Personal Vault/**"` to skip it

### Transfer Speed Issues

If transfers are slow:
- Check your internet connection
- Try `--transfers 4` to increase parallel transfers (default is 4)
- Use `--bwlimit` if you're saturating your connection and need to limit speed

### Verifying File Integrity

After large transfers, verify everything copied correctly:

```bash
rclone check /media/mark/Ext_T7_lpthp/OneDrive onedrive:/ --one-way
```

## Additional Resources

- Official rclone docs: https://rclone.org/
- OneDrive-specific docs: https://rclone.org/onedrive/
- rclone forum: https://forum.rclone.org/

## Summary

- Rclone is installed and configured to access your OneDrive
- Use `rclone copy` for safe one-way transfers
- Use `rclone sync` with caution (can delete files)
- Rclone does NOT auto-sync - you must run it manually
- Your initial download to external drive is complete (125 files, ~150 MB)
- Personal Vault cannot be accessed via rclone (expected limitation)

For your use case (one-time bulk download), rclone is perfect. If you later need continuous sync like Dropbox, consider the abraunegg/onedrive client instead.
