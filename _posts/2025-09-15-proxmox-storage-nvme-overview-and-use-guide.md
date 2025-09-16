---
layout: post
title: "Proxmox server NVMe Storage Overview and Guide"
draft: false
date: 2025-09-15
description:  After almost losing my Proxmox server - thank God for recent backups and snapshots - I decided to document how the storage was set up this time, which was more logical than the initial setup.  Two internal NVMe SSDs and one external SSD.  General outline of where to host the OS, backups, VMs, and external data used by the VMs, including how to mount it into the Docker containers correctly.  
---

## Proxmox Server NVMe Storage Overview Guide

Q: I have a Proxmox server with two internal storage NVMe SSDs and I'd like to see an overview of what's on all of them. How do I do that?

To get a full overview of what's on your **Proxmox server's internal NVMe SSDs**, you'll want to check:

1. **Device details** (names, sizes, mount points)
2. **Partitioning and filesystem usage**
3. **Storage configuration** in Proxmox (LVM, ZFS, directories, etc.)
4. **Mounted volumes and disk usage**

Here's how to get a full picture — step-by-step:

## 🔍 Step-by-Step: View What's on Your NVMe Drives

###   1. List All Block Devices

```sh
lsblk -o NAME,MODEL,SIZE,FSTYPE,MOUNTPOINT,LABEL
```

This shows:
- All disks and partitions
- Filesystem type (ext4, zfs_member, LVM2_member, etc.)
- Mount points like `/`, `/var/lib/vz`, etc.

Sample Output:

```
NAME        MODEL        SIZE FSTYPE      MOUNTPOINT
nvme0n1     Samsung 980  1.0T
├─nvme0n1p1               512M vfat       /boot/efi
├─nvme0n1p2               1.0T LVM2_member
│ └─pve-root              96G ext4       /
│ └─pve-data             900G ext4       /var/lib/vz
nvme1n1     WD SN850X    1.0T zfs_member
```

###   2. Check Mounted Filesystems and Disk Usage

```sh
df -hT
```

- Shows what partitions are mounted, where, and how full they are.
- Useful to see how much space is used on each mount point.

###   3. Identify LVM Volumes (Common in Proxmox)

If you're using LVM (default for non-ZFS installs):

```sh
vgs       # Show volume groups
lvs       # Show logical volumes
lsblk     # Again — LVs show here too
```

Example output:

```
VG  #PV #LV  Attr    VSize   VFree
pve   1   3   wz--n-  931.00g 20.00g
```

###   4. Check ZFS Pools (If You're Using ZFS)

```sh
zpool status
zfs list
```

- `zpool status` shows health and disk layout
- `zfs list` shows datasets, mount points, and space usage

Example:

```
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH
rpool   931G   150G   781G        -         -     5%    16%  1.00x  ONLINE
```

###   5. See Proxmox Storage Configuration

Proxmox manages storage via `/etc/pve/storage.cfg`. Check it with:

```sh
cat /etc/pve/storage.cfg
```

This file shows how storage is defined in the Proxmox UI — LVM, ZFS pools, directories, etc.

Example:

```
dir: local
    path /var/lib/vz
    content iso,backup,vztmpl,container,images

zfspool: local-zfs
    pool rpool/data
    content images,rootdir
    sparse 1
```

###   6. See Which VMs/CTs Are Using Which Disk

```sh
qm list                 # List all VMs
pct list                # List all LXC containers
```

Then inspect individual VMs:

```sh
qm config <vmid>
```

Look for lines like:

```
scsi0: local-zfs:vm-100-disk-0,size=32G
```

Or for containers:

```sh
pct config <ctid>
```

###   Bonus: See SMART Info for NVMe Drives

```sh
smartctl -a /dev/nvme0n1
smartctl -a /dev/nvme1n1
```

You may need to install `smartmontools` first:

```sh
apt install smartmontools
```

## 📦 Summary

| Goal | Command |
|------|---------|
| View disks, partitions, mount points | `lsblk -o NAME,MODEL,SIZE,FSTYPE,MOUNTPOINT,LABEL` |
| View disk usage | `df -hT` |
| View LVM volumes | `vgs`, `lvs` |
| View ZFS pools | `zpool status`, `zfs list` |
| View storage config | `cat /etc/pve/storage.cfg` |
| View VM/CT storage usage | `qm config <vmid>`, `pct config <ctid>` |
| SMART info for NVMe | `smartctl -a /dev/nvmeXn1` |