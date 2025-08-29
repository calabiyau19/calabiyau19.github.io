---
layout: post
title: "Linux Maintenance Tasks"
date:  2025-03-17
description:  Keeping your Linux system clean and up to date is essential. Here are some easy maintenance routines to follow.  Please check out number eleven for a good app to help you manage files easily.  Hint. ncdu
---
8/01/2025 UPDATE:  Wrote a weekly maintenance script (with help). Copied it to a folder called ~/Scripts on each Ubuntu and Debian machine.  Using Tabby, I can access each machine quickly via ssh (key pairs set up already) and run this copied command, ~/Scripts/ubuntu-maintenance.sh, which runs the script that does all the updating and cleanup functions.  Since each 'machine' is in its own VM I can open a new tab (to a new machine) while the script is still running on the previous machine.  So, can run through all the machines weekly in less than 10 minutes.  There are great products to help automate this but for ten minutes once a week the speed of this method is fine.  
The maintenance script is at the end of this post

TL:DR   Always run at least these weekly, if not more often.

```sh
sudo apt update && sudo apt upgrade -y
```

```sh
sudo apt autoremove && sudo apt clean && sudo apt autoclean
```

This shows how much disk space you have used and have left

```sh
df -h
```

This guide outlines 11 additional steps to keep your Linux system clean, optimized, and clutter-free. Each step includes explanations of the commands and their purpose, along with a suggested schedule for maintenance.

1) List Removed Packages with Leftover Configuration Files
Find packages that have been uninstalled but still have leftover configuration files on your system.

```sh
dpkg -l | grep ^rc
```

Explanation: This command checks your system for packages in the "rc" state (removed but with configuration files left). The `grep ^rc` filters these results.

Schedule: Run this monthly to identify leftover configuration files.

2) Purge Leftover Configuration Files
Remove all leftover configuration files from uninstalled packages.

```sh
dpkg -l | awk '/^rc/{print $2}' | xargs sudo dpkg --purge
```

Explanation: This command uses `awk` to extract the names of packages in the "rc" state and passes them to `dpkg --purge` to remove their configuration files.

Schedule: Run this immediately after Step 1.

3) Remove Specific Unused Packages
If you know certain packages are no longer needed, remove them along with their configuration files.

```sh
sudo apt-get remove --purge <package-name>
```

Explanation: Replace <package-name> with the package you want to remove. The --purge flag ensures both the package and its configuration files are deleted.

Schedule: Run as needed when you identify unused packages.

4) Remove Orphaned Packages
Clear out packages that were automatically installed as dependencies but are no longer needed.

```sh
sudo apt-get autoremove --purge
```

Explanation: The `autoremove` command removes orphaned packages, and `--purge` ensures their configuration files are also deleted.

Schedule: Run weekly or biweekly.

5) Clean APT Cache
Clear the cache of downloaded package files to free up space.

```sh
sudo apt-get clean
```

Explanation: This command removes downloaded package files stored in /var/cache/apt/archives. These files are not needed after installation.

Schedule: Run monthly or when low on disk space.

6) Manage System Logs

First, display  the siz of your logs.

```sh
journalctl --disk-usage
```

Or the following, which also gives the log location.

```sh
du -sh /var/log/journal
```

By default, on Linux Mint 22 logs are in most cases capped at a certain size.
Running the following  command ensures that your system logs don't exceed 99 MB in size.

```sh
sudo journalctl --vacuum-size=99M
```

Schedule: Run monthly or when logs grow too large


To make this automatic, first open the journald.config file.

```sh
sudo nano /etc/systemd/journald.conf
```

Then change or add:
SystemMaxUse=99M
And then Save the file and apply the change.

```sh
sudo systemctl restart systemd-journald
```

7) Find and Remove Large Files
Identify large files or directories that consume significant disk space.

```sh
sudo du -ahx / 2>/dev/null | sort -h | tail -n 20
```
Better

```sh
sudo du -hx -d 2 / 2>/dev/null | sort -h | tail -n 20
```

The following command displays large files but skips /mnt - which may cause the command to appear to hang when all it might be doing is looking into /mnt and then into whatever is mounted there such as external drives, network shares, etc.

```sh
[ -d /home/mark/mnt ] && echo "⚠️ Skipping /home/mark/mnt (external drive detected)" >&2
sudo du -hx --exclude=/home/mark/mnt -d 1 /home/mark 2>/dev/null | sort -h
```


8) Clear Cache Files
Remove user-specific cache files to free up space.

```sh
rm -rf ~/.cache/*
```

Explanation: This command removes cache files in your home directory. Cache files are safe to delete as they are recreated when needed.

Schedule: Run monthly.

9) Uninstall Unused Snap Packages (Optional)
Remove Snap packages and old versions you no longer use.

```sh
sudo snap remove <snap-name>
```

```sh
sudo snap remove --purge
```
Explanation: Use `snap remove` to uninstall Snap packages and `--purge` to remove their data completely.

Schedule: Run as needed.

10) Run Bleachbit last
Run Bleachbit – it is installed

11)  ## OPTIONAL:  Much better file system manager for cleanup purposes but must install  ncdu first
ncdu is NCurses Disk Usage.  Like du but much faster and graphical and interactive

```sh
sudo apt update && sudo apt install ncdu -y
```

Then various scans:
Root scan

```sh
sudo ncdu /
```
Specific folder scan

```sh
ncdu /home
```

Ignore mounted drives (faster scan)

```sh
sudo ncdu -x /
```
BONUS CONTENT:

Set up reminders for weekly, monthly and quarterly with notifications and links to this article

Open crontab to add reminders.

```sh
crontab -e
```

Add the following lines and change the document location to match your own

❯ #Weekly maintenance task reminder                                                                            ─╯
  0 9 * * 1 DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Weekly System Maintenance" "Reminder: Perform your weekly system maintenance tasks! Check your guide at https://calabiyau19.github.io/posts/Linux-Maintenance-Tasks/"'

  #Monthly maintenance task reminder
  0 9 1 * * DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Monthly System Maintenance" "Reminder: Perform your monthly system maintenance tasks! Check your guide at https://calabiyau19.github.io/posts/Linux-Maintenance-Tasks/"'

  #Quarterly maintenance task reminder
  0 9 1 1,4,7,10 * DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/env fish -c 'notify-send "Quarterly System Maintenance" "Reminder: Perform your quarterly system maintenance tasks! Check your guide at https://calabiyau19.github.io/posts/Linux-Maintenance-Tasks/"'

You will get a popup reminder (tested in Linux MInt) to perform the maintenance tasks as outlined

Schedule: Run monthly.



Maintenance Script for Ubuntu + Debian machines


#!/bin/bash

# Ubuntu Weekly Maintenance Script
# Run manually with: sudo ~/Scripts/ubuntu-maintenance.sh

set -euo pipefail

# Check if running as root
if [[ "$EUID" -ne 0 ]]; then
  echo "Please run as root: sudo $0"
  exit 1
fi

echo "=== Starting Ubuntu Maintenance ==="

# Update and upgrade
echo ">>> Running apt update and upgrade..."
apt update && apt upgrade -y

# Cleanup tasks
echo ">>> Running autoremove, clean, and autoclean..."
apt autoremove -y && apt clean && apt autoclean -y

# Remove orphaned config files
echo ">>> Purging orphaned package config files..."
dpkg -l | awk '/^rc/ {print $2}' | xargs -r dpkg --purge

# Extra purge cleanup
echo ">>> Running additional autoremove --purge..."
apt-get autoremove --purge -y

# Clean apt cache
echo ">>> Final apt-get clean..."
apt-get clean

# Vacuum logs
echo ">>> Vacuuming journal logs to 99M..."
journalctl --vacuum-size=99M

# Clear user's cache
echo ">>> Clearing user cache for mark..."
rm -rf /home/mark/.cache/*

echo "=== Maintenance Complete ==="
