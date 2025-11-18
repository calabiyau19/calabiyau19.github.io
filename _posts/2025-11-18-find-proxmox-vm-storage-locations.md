---
layout: post
title: "Find Proxmox VM data storage locations across multiple drives"
draft: false
date: 2025-11-18
description: "This documents the physical location of the images stored on my Proxmox server for the application called Immich running in a VM on that same host.  The physical location is tricky since the app is on one ssd, backups on another ssd and data on yet another ssd.  I did this for me when I go looking for the physcial location every 6 months and forget how to do it."
---
Data location in docker-compose.yml(via .env) is /mnt/immich-storage.

## Finding Physical Storage Location of VM Data on Proxmox

## Step 1: Identify the VM

```sh
qm list | grep -i immich
```

**Result:** VMID 1102

## Step 2: View VM configuration

```sh
qm config 1102
```

**Result:** Shows all attached disks and their storage locations

## Step 3: Identify the data disk from the config

From the Proxmox web UI Hardware tab or the config output, identify which disk contains your data.

**Result:** `scsi1 = nvme2tb-vm:1102/vm-1102-disk-1.qcow2 (400G)`

## Step 4: Get the physical path

```sh
pvesm path nvme2tb-vm:1102/vm-1102-disk-1.qcow2
```

**Result:** `/mnt/nvme2tb/vm-storage/images/1102/vm-1102-disk-1.qcow2`

## Step 5: Verify the file exists

```sh
ls -lh /mnt/nvme2tb/vm-storage/images/1102/vm-1102-disk-1.qcow2
```

**Result:** Confirms 401G qcow2 file containing your photos

## Step 6: Verify backup includes this disk

```sh
pvesm status | grep backup
```

**Result:** Shows backup storage location

```sh
cat /etc/pve/storage.cfg | grep -A 5 nvme1tb-backup
```

**Result:** Shows backup path is `/mnt/nvme1tb/backup`

```sh
ls -lh /mnt/nvme1tb/backup/dump/ | tail -3
```

**Result:** Shows most recent backup

```sh
cat /mnt/nvme1tb/backup/dump/vzdump-qemu-1102-2025_11_18-08_02_12.log | grep "include disk"
```

**Result:** Confirms both disks backed up.
