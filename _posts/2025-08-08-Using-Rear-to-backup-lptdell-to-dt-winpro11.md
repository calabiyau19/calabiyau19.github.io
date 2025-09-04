---
layout: post
title: "Using ReaR to backup a linux pc to a shared windows drive in one step"
draft: false
date:  2025-08-08
description: The object of this exercise was to fully backup lpt-dell to a bootable ISO file on a windows drive shared on the network that is designated for backups.  With this, lpt-dell could be brought back to the exact same condition as when the ISO was created with ReaR - which stands for Relax and Recover.
---



**Relax-and-Recover (ReaR) Backup Summary for lpt-dell to Windows SMB Share via SSHFS**

**System Being Backed Up:**

* Hostname: lpt-dell
* OS: Xubuntu (with Fish shell)
* Goal: Full system backup + bootable ISO stored on network share (Windows drive E:)

**Backup Destination:**

* Machine: Windows 11 Pro at 192.168.1.200
* Share path: E:\rear-backups-lpt-dell
* Access via: SSHFS from lpt-dell as user mark

---

**Final Verified ReaR Configuration**

/etc/rear/local.conf:

ini
OUTPUT=ISO
BACKUP=NETFS
BACKUP_URL=file:///home/mark/mnt/rear-backups-lpt-dell


---

**Working Backup Procedure**

1. **Ensure SSH access to Windows is working:**

```sh
ssh mark@192.168.1.200
```

2. **Unmount previous mount if needed:**

```sh
fusermount -u ~/mnt/rear-backups-lpt-dell
```

3. **Mount the Windows backup folder with proper permissions:**

```sh
sshfs -o allow_other mark@192.168.1.200:/E:/rear-backups-lpt-dell ~/mnt/rear-backups-lpt-dell
```


4. **Clean up any stale files (optional but recommended):**

```sh```
rm ~/mnt/rear-backups-lpt-dell/rear-lpt-dell.iso
```

5. **Create the backup folder on the share (if not already present):**

```sh
mkdir -p ~/mnt/rear-backups-lpt-dell/lpt-dell
```


6. **Run ReaR backup (as root):**

```sh
sudo rear -v mkbackup
```

**Outcome:**

* ISO created at /var/lib/rear/output/rear-lpt-dell.iso
* Backup archive written to /home/mark/mnt/rear-backups-lpt-dell/lpt-dell/backup.tar.gz

---

**Notes:**

* ReaR rejects BACKUP\_URL paths that reside on / partition. Using mounted SSHFS folder bypasses this.
* The ISO and backup are not stored together by default; ISO must be copied separately if needed.
* Root access is required to run rear mkbackup and access mounted destinations.

**Optional Cleanup:**
To remove temporary ReaR data from the local system:

```sh
sudo rm -rf /var/tmp/rear.*
sudo rm -f /var/lib/rear/output/rear-lpt-dell.iso
```


**Status:** Fully verified as working.
