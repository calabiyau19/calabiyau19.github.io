---
layout: post
draft: false
title: "Creating Network Storage on External Drive and YouTube Download System"
date: 2025-12-21
description: "This guide documents setting up a centralized Samba network storage server on Proxmox using an external SSD"
categories: [networking, samba, windows]
---

## Creating Network Storage on External Drive and YouTube Download System

### Overview

This guide documents setting up a centralized Samba network storage server on Proxmox using an external SSD, configuring it for universal access across Linux, Windows, and iOS devices, fixing Navidrome music streaming, and setting up yt-dlp for downloading YouTube videos directly to network storage with mobile streaming capability.

### Infrastructure Assessment

**Proxmox Hosts:**

- .125 (nuc5): 60 GB RAM, 29 GB available
- .121 (nuc3): 16 GB RAM, 2.6 GB available

**Storage on .125:**

- 1 TB NVMe (nvme1tb): Backups
- 2 TB internal NVMe (nvme2tb): Applications and VMs
- 2 TB external SSD (Ext_T7_SSD): 1.7 TB available - chosen for network storage

Commands run on both Proxmox hosts:

```sh
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE
df -h
free -h
top -bn1 | head -20
qm list
```

### Cleaning the External SSD

Check current contents on .125:

```sh
ls -lh /mnt/Ext_T7_ssd/
du -sh /mnt/Ext_T7_ssd/*
```

Move music to permanent location:

```sh
mkdir -p /mnt/nvme2tb/music
mv /mnt/Ext_T7_ssd/music/* /mnt/nvme2tb/music/
```

Delete old files:

```sh
rm -rf /mnt/Ext_T7_ssd/nvme2tb-backup/
rm -rf /mnt/Ext_T7_ssd/dump/
rm -rf /mnt/Ext_T7_ssd/music/
```

Verify clean:

```sh
ls -lh /mnt/Ext_T7_ssd/
df -h /mnt/Ext_T7_ssd
```

### Creating Samba Storage VM

Clone Ubuntu template on .125:

```sh
qm clone 890 1106 --name samba-storage-vm --full
```

Start VM and get IP address (192.168.1.167)

Update hostname on VM:

```sh
ssh mark@192.168.1.167
hostnamectl set-hostname samba-storage-vm
hostname
```

### Passing External Drive to VM

On Proxmox host .125, identify the drive:

```sh
ls -l /dev/disk/by-id/ | grep -i sda
```

Update NFS exports (remove old Ext_T7_SSD shares):

```sh
nano /etc/exports
```

Changed to export music from new location:

```
/mnt/nvme2tb/music 192.168.1.0/24(ro,sync,no_subtree_check)
```

Reload exports:

```sh
exportfs -ra
```

Unmount drive from Proxmox host:

```sh
umount /mnt/Ext_T7_ssd
```

Attach drive to VM:

```sh
qm set 1106 -scsi1 /dev/disk/by-id/usb-Samsung_PSSD_T7_S7MPNS0XA03986E-0:0
```

Verify attachment:

```sh
qm config 1106 | grep scsi
```

### Mounting Drive Inside VM

SSH into samba-storage-vm:

```sh
ssh mark@192.168.1.167
```

Check for new disk:

```sh
lsblk
```

Create mount point:

```sh
sudo mkdir -p /mnt/storage
```

Mount the drive:

```sh
sudo mount /dev/sdb1 /mnt/storage
```

Verify mount:

```sh
df -h /mnt/storage
```

Get UUID for permanent mount:

```sh
sudo blkid /dev/sdb1
```

Add to fstab:

```sh
sudo nano /etc/fstab
```

Added:

```
UUID=18e4bb73-2214-486f-a4da-0efef03a4eec /mnt/storage ext4 defaults 0 2
```

Test fstab:

```sh
sudo mount -a
```

### Installing and Configuring Samba

Update and install Samba:

```sh
sudo apt update
sudo apt upgrade -y
sudo apt install samba -y
```

Backup original config:

```sh
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

Configure Samba share:

```sh
sudo nano /etc/samba/smb.conf
```

Added at bottom:

```
[storage]
   path = /mnt/storage
   browseable = yes
   read only = no
   create mask = 0775
   directory mask = 0775
   valid users = mark
```

Set Samba password:

```sh
sudo smbpasswd -a mark
```

Restart Samba:

```sh
sudo systemctl restart smbd
```

Verify running:

```sh
sudo systemctl status smbd
```

### Testing Access from Linux Laptop

On Linux Mint laptop, access via file manager:

```
smb://192.168.1.167/storage
```

Login with username: mark and Samba password

Test file creation and verify read/write access

### Setting Up iOS Access

Install Infuse from App Store

Add SMB share in Infuse:

- Name: Network Storage
- Protocol: SMB
- Address: 192.168.1.167/storage
- Username: mark
- Password: [Samba password]

Enable background audio in Infuse settings → Playback → Background Audio: ON

### Fixing Navidrome Music Access

Update NFS mount on Navidrome VM (.152):

```sh
ssh mark@192.168.1.152
```

Edit fstab:

```sh
sudo nano /etc/fstab
```

Changed:

```
192.168.1.125:/mnt/nvme2tb/music /mnt/navidrome/music nfs ro,nofail,x-systemd.automount,_netdev 0 0
```

Stop Navidrome:

```sh
cd ~/navidrome
docker compose down
```

On Proxmox host .125, verify NFS server running:

```sh
systemctl status nfs-server
```

Start if needed:

```sh
systemctl start nfs-server
```

Verify NFS daemons:

```sh
rpcinfo -p | grep nfs
```

Verify export active:

```sh
exportfs -v
```

Back on Navidrome VM, verify can see export:

```sh
showmount -e 192.168.1.125
```

Manually mount:

```sh
sudo mount -t nfs 192.168.1.125:/mnt/nvme2tb/music /mnt/navidrome/music
```

Verify mount:

```sh
mount | grep navidrome
```

Test access:

```sh
ls -lh /mnt/navidrome/music/ | head -20
```

Start Navidrome:

```sh
docker compose up -d
```

### Setting Up yt-dlp Network Downloads

On Linux Mint laptop, create mount point:

```sh
sudo mkdir -p /mnt/network-storage
```

Create credentials file:

```sh
nano ~/.smbcreds
```

Contents:

```
username=mark
password=YOUR_SAMBA_PASSWORD
```

Secure credentials:

```sh
chmod 600 ~/.smbcreds
```

Add to fstab:

```sh
sudo nano /etc/fstab
```

Added:

```
//192.168.1.167/storage /mnt/network-storage cifs credentials=/home/mark/.smbcreds,uid=1000,gid=1000 0 0
```

Reload systemd:

```sh
sudo systemctl daemon-reload
```

Mount the share:

```sh
sudo mount /mnt/network-storage
```

Verify mount:

```sh
mount | grep network-storage
```

Test access:

```sh
ls /mnt/network-storage
```

Update yt-dlp alias:

```sh
nano ~/.config/fish/config.fish
```

Changed from:

```
alias yt-dlp-save="yt-dlp --write-sub --write-auto-sub --sub-lang 'en.*' --convert-subs srt --restrict-filenames -o '/media/mark/Ext_T7_lpthp/YouTube-downloads/%(uploader)s/%(title)s/%(title)s.%(ext)s'"
```

To:

```
alias yt-dlp-save="yt-dlp --write-sub --write-auto-sub --sub-lang 'en.*' --convert-subs srt --restrict-filenames -o '/mnt/network-storage/YouTube-downloads/%(uploader)s/%(title)s/%(title)s.%(ext)s'"
```

Reload fish config:

```sh
source ~/.config/fish/config.fish
```

Test download:

```sh
yt-dlp-save https://www.youtube.com/watch?v=dQw4w9WgXcQ
```

Verify file on network storage:

```sh
ls -lh /mnt/network-storage/YouTube-downloads/
```

### Final Configuration Summary

**Network Storage Server (VM 1106 on .125):**

- IP: 192.168.1.167
- Storage: 1.7 TB external SSD at /mnt/storage
- Samba share name: storage
- Access: \\192.168.1.167\storage (Windows) or smb://192.168.1.167/storage (Linux/Mac)

**Access Methods:**

- Linux: Mounted at /mnt/network-storage
- Windows: \\192.168.1.167\storage
- iOS: Infuse app with SMB connection

**yt-dlp Integration:**

- Downloads go directly to /mnt/network-storage/YouTube-downloads/
- Organized by uploader/title structure
- Accessible from all devices

**Navidrome Music Streaming:**

- Music location: /mnt/nvme2tb/music on Proxmox host .125
- Shared via NFS to Navidrome VM
- Mounted at /mnt/navidrome/music

**Mobile Streaming:**

- Use Infuse app on iOS
- Background audio enabled for screen-off listening
- Access via SMB to network storage

### Key Lessons Learned

- Always verify NFS server is running before troubleshooting client-side mount issues
- Check basic infrastructure (is service running?) before complex troubleshooting
- Unmounting actively-exported filesystems can stop NFS server
- Systemd automounts require daemon-reload after fstab changes
- Simple solutions (like rebooting) often work better than complex systemd commands

### Next Steps

- Set up Tailscale for remote access to network storage
- Configure automatic backups of network storage
- Consider additional Samba shares for specific purposes
