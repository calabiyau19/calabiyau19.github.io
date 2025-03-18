---
layout: post
title: "Linux Maintenance Tasks"
date:  2025-03-17
---


Comprehensive System Cleanup Guide
Start with the basics

```sh
sudo apt update && sudo apt upgrade -y
```

```sh
sudo apt autoremove && sudo apt clean && sudo apt autoclean
```

```sh
df -h
```

```sh

sudo du -h / 2>/dev/null | sort -h | tail -n 20
```

This guide outlines 11 essential steps to keep your Linux system clean, optimized, and clutter-free. Each step includes explanations of the commands and their purpose, along with a suggested schedule for maintenance.

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

1. Remove Specific Unused Packages
If you know certain packages are no longer needed, remove them along with their configuration files.

```sh
sudo apt-get remove --purge <package-name>
```

Explanation: Replace <package-name> with the package you want to remove. The --purge flag ensures both the package and its configuration files are deleted.

Schedule: Run as needed when you identify unused packages.

4. Remove Orphaned Packages
Clear out packages that were automatically installed as dependencies but are no longer needed.

```sh
sudo apt-get autoremove --purge
```

Explanation: The `autoremove` command removes orphaned packages, and `--purge` ensures their configuration files are also deleted.

Schedule: Run weekly or biweekly.

5. Clean APT Cache
Clear the cache of downloaded package files to free up space.

```sh
sudo apt-get clean
```

Explanation: This command removes downloaded package files stored in /var/cache/apt/archives. These files are not needed after installation.

Schedule: Run monthly or when low on disk space.

6. Manage System Logs
Limit the size of system logs to prevent them from consuming too much space.

```sh
sudo journalctl –vacuum-size=100M
```

```sh
sudo journalctl --vacuum-time=7d
```

Explanation: This command ensures that your system logs don't exceed 100 MB in size.

Schedule: Run monthly or when logs grow excessively.

7. Find and Remove Large Files
Identify large files or directories that consume significant disk space.

```sh
sudo du -h / --max-depth=1 | sort -hr | head -10```
```

```sh
du -h / | sort -h | tail -n 20  ←--needs sudo
```

```sh
sudo du -h / 2>/dev/null | sort -h | tail -n 20
```

Explanation: This command uses `du` to calculate the size of directories, sorts them by size, and displays the top 10 largest directories.

Schedule: Run quarterly or when space is low.

8. Clear Cache Files
Remove user-specific cache files to free up space.

```sh
rm -rf ~/.cache/*
```
Explanation: This command removes cache files in your home directory. Cache files are safe to delete as they are recreated when needed.

Schedule: Run monthly.

9. Uninstall Unused Snap Packages (Optional)
Remove Snap packages and old versions you no longer use.

```sh
sudo snap remove <snap-name>
```

```sh
sudo snap remove --purge
```
Explanation: Use `snap remove` to uninstall Snap packages and `--purge` to remove their data completely.

Schedule: Run as needed.

10. Perform Regular System Updates
Ensure your system is updated with the latest packages and security patches.

```sh
sudo apt update && sudo apt upgrade
```

Explanation: `apt update` refreshes the package index, and `apt upgrade` installs the latest versions of packages.

11. Run Bleachbit last
Run Bleachbit – it is installed

EXTRAS:
╰─ 
```sh
sudo ncdu --exclude /media --exclude /mnt --exclude /proc --exclude /sys --exclude /dev /BACKUPS\ TEMP/
```

-this is a new(old) command line program called ncdu


Schedule: Run weekly.

Suggested Maintenance Schedule
Weekly: Steps 4, 10

Monthly: Steps 1, 2, 5, 6, 8

Quarterly: Steps 7

As Needed: Steps 3, 9
