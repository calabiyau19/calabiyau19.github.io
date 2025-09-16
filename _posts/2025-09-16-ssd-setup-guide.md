---
layout: post
title: "SSD setup guide for Linux"
draft: false
date: 2025-09-16
description:  "Still documenting how to attach a external SSD (or could be internal) on a linux machine or server and how to initially set it up for basic use.  Then, how to get a Docker app running in a VM on a Proxmox server to access and use data on the drive. Example: all music used by Navidrome is on the SSD, or all audiobooks used by Audiobookshelf or digital books, etc." 
---


# External SSD Setup Guide for Proxmox and Docker

Scenario: All the steps involve plugging in a new external storage unit - specifically, a 2 TB SSD you've just unwrapped and plugged in, with no additional setup.

The process involves several steps: making the USB drive usable as an ext4 storage device (with only one partition), mounting it, and then accessing it from within a Docker container, which is in a VM on the Proxmox server to which it is attached. This is the exact way the SSD connected to Proxmox-nuc5 was set up for Navidrome's music.

## Part 1: On the Proxmox Host

### Step 1: Identify the new USB SSD

```sh
lsblk
```

Look for something like `/dev/sdX` with no mount point and ~2TB size.

### Step 2: Partition the disk (single partition, ext4)

```sh
sudo parted /dev/sdX --script mklabel gpt mkpart primary ext4 0% 100%
```

### Step 3: Format the partition

```sh
sudo mkfs.ext4 /dev/sdX1
```

### Step 4: Create a mount point and mount it

```sh
sudo mkdir -p /mnt/Ext_New_SSD
sudo mount /dev/sdX1 /mnt/Ext_New_SSD
```

### Step 5: Make the mount persistent

```sh
sudo blkid
```

Copy the UUID for `/dev/sdX1`, then:

```sh
sudo nano /etc/fstab
```

Add this line (replace `UUID=xxxxx`):

```sh
UUID=xxxxx /mnt/Ext_New_SSD ext4 defaults 0 2
```

Then mount:

```sh
sudo mount -a
```

### Step 6: Create music folder & set permissions

```sh
sudo mkdir -p /mnt/Ext_New_SSD/music
sudo chown root:root /mnt/Ext_New_SSD/music
sudo chmod 755 /mnt/Ext_New_SSD/music
```

### Step 7: Export via NFS

Install NFS server if needed:

```sh
sudo apt update
sudo apt install nfs-kernel-server
```

Edit exports:

```sh
sudo nano /etc/exports
```

Add:

```sh
/mnt/Ext_New_SSD/music 192.168.1.0/24(ro,sync,no_subtree_check)
```

Apply and restart:

```sh
sudo exportfs -ra
sudo systemctl restart nfs-server rpcbind
```

## Part 2: On the Ubuntu VM (e.g. navidrome-vm)

### Step 8: Install NFS client

```sh
sudo apt update
sudo apt install nfs-common
```

### Step 9: Create the mount point

```sh
sudo mkdir -p /mnt/navidrome/music
```

### Step 10: Mount the share (manual test)

```sh
sudo mount -t nfs 192.168.1.125:/mnt/Ext_New_SSD/music /mnt/navidrome/music
```

Verify:

```sh
ls -lah /mnt/navidrome/music
```

### Step 11: Make it persistent

```sh
sudo nano /etc/fstab
```

Add:

```sh
192.168.1.125:/mnt/Ext_New_SSD/music /mnt/navidrome/music nfs ro,nofail,x-systemd.automount,_netdev 0 0
```

Mount all to test:

```sh
sudo mount -a
```

## Part 3: Back on the Ubuntu VM – Restart Docker Container

### Step 12: Ensure docker-compose is pointing to the right music path

Update `docker-compose.yml`:

```sh
volumes:
  - ${HOME}/navidrome/config:/data
  - /mnt/navidrome/music:/music:ro
```

Then:

```sh
cd ~/navidrome
docker compose down
docker compose up -d
```