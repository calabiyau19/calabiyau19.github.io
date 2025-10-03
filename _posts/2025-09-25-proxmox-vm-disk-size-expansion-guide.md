---
title: Extending the size of a VM on Proxmox
date: 2025-09-25
description: "You spun up a VM, added applications to it and realized later on you made it too small since you are using it more and more and it is growing.  Possibly you got warnings about it being critically short of space.  This article shows how to make the VM larger than what you inititally set it up with."
---

NOTE: MAKE A BACKUP OR TAKE A SNAPSHOT BEFORE ATTEMPTING TO RESIZE THE DISK!

## VM Disk Expansion: 15 GB to 50 GB Complete Guide

This first command happens inside Proxmox's GUI and you first select the VM you need to make larger.  Once you select it, look in the center panel for the VM's hardware option and continue with the below instructions from there.

### 1. Proxmox GUI - Resize Virtual Disk

**Command:** Hardware → Select Hard Disk → Resize disk → Add desired GB → Confirm

- **What it does:** Increases the size of the virtual disk file on the Proxmox host
- **Why needed:** This expands the virtual disk container from 15 GB to 50 GB at the hypervisor level, but the VM doesn't know about the extra space yet
- **Result:** The virtual disk grows to 50 GB, but all storage layers inside the VM still think it's 15 GB

---

### Everything Below Happens Inside the Ubuntu VM

### 2. Fix GPT Table and Extend Partition

```sh
sudo parted /dev/sda resizepart 3 100%
```

- **What it does:** Updates the partition table to recognize the new disk size and extends partition 3 to use all available space
- **Why needed:** The VM's partition table still thinks the disk is 15 GB. The partition boundaries need to be updated to access the newly available space from Proxmox
- **Result:** Partition `sda3` expands from ~13 GB to ~48 GB

### 3. Extend LVM Physical Volume

```sh
sudo pvresize /dev/sda3
```

- **What it does:** Tells Ubuntu's LVM system that the underlying partition is now larger
- **Why needed:** Ubuntu uses LVM (Logical Volume Manager) for storage management. The LVM Physical Volume layer needs to scan the partition and recognize the additional space
- **Result:** The LVM Physical Volume now knows it has ~48 GB of space available instead of ~13 GB

### 4. Extend LVM Logical Volume

```sh
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

- **What it does:** Expands the logical volume to consume all newly available space in the volume group
- **Why needed:** Even though the Physical Volume knows about the extra space, the Logical Volume (the virtual partition that contains your file system) is still set to its original size
- **Result:** The logical volume `ubuntu-lv` grows from ~13 GB to ~48 GB

### 5. Extend File system

```sh
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

- **What it does:** Resizes the ext4 file system to fill the entire logical volume
- **Why needed:** The file system manages your actual files and directories. It still thinks it's operating in the original smaller space and won't use the extra room until explicitly told to expand
- **Result:** Your usable file system space grows from ~13 GB to ~48 GB

### 6. Verify Expansion

```sh
df -h` or `lsblk
```

- **What to expect:** `df -h` should show ~48 GB total space with much lower usage percentage

---

### Storage Layer Summary

Each step works on a different layer of the storage stack inside the VM:
**Proxmox Virtual Disk → Ubuntu Partition → LVM Physical Volume → LVM Logical Volume → ext4 File system**

Each layer needs to be explicitly told about the expansion in this exact sequence.
