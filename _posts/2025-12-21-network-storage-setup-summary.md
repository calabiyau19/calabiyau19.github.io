---
layout: default
title: "Network Storage Setup: External SSD on Proxmox with Samba"
date: 2025-12-21
categories: [networking, samba, windows]
---



## Network Storage Setup: External SSD on Proxmox with Samba

### Overview

This guide explains how to connect an external SSD to a Proxmox host, pass it through to a VM, configure Samba for network sharing, and access it from Linux and Windows clients.

### Step 1: Connect and Identify the External SSD on Proxmox Host

Plug the external SSD into the Proxmox server's USB port.

Identify the drive:

```sh
ls -l /dev/disk/by-id/ | grep -i sda
```

Note the full device ID (e.g., `usb-Samsung_PSSD_T7_S7MPNS0XA03986E-0:0`)

### Step 2: Create or Clone a VM for Storage

Create a new Ubuntu VM or clone from an existing template:

```sh
qm clone 890 1106 --name samba-storage-vm --full
```

Start the VM and note its IP address (e.g., 192.168.1.167)

### Step 3: Pass the External SSD to the VM

If the drive is currently mounted on the Proxmox host, unmount it:

```sh
umount /mnt/your-mount-point
```

Attach the drive to the VM using its device ID:

```sh
qm set 1106 -scsi1 /dev/disk/by-id/usb-Samsung_PSSD_T7_S7MPNS0XA03986E-0:0
```

Verify the attachment:

```sh
qm config 1106 | grep scsi
```

### Step 4: Mount the Drive Inside the VM

SSH into the VM:

```sh
ssh mark@192.168.1.167
```

Check for the new disk:

```sh
lsblk
```

Create a mount point:

```sh
sudo mkdir -p /mnt/storage
```

Mount the drive (assuming it appears as /dev/sdb1):

```sh
sudo mount /dev/sdb1 /mnt/storage
```

Get the UUID for permanent mounting:

```sh
sudo blkid /dev/sdb1
```

Add to /etc/stab for automatic mounting on boot:

```sh
sudo nano /etc/fstab
```

Add this line (replace UUID with your actual UUID):

```
UUID=18e4bb73-2214-486f-a4da-0efef03a4eec /mnt/storage ext4 defaults 0 2
```

Test the fstab entry:

```sh
sudo mount -a
```

### Step 5: Install and Configure Samba on the VM

Update and install Samba:

```sh
sudo apt update
sudo apt upgrade -y
sudo apt install samba -y
```

Backup the original Samba configuration:

```sh
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

Edit the Samba configuration:

```sh
sudo nano /etc/samba/smb.conf
```

Add this at the bottom of the file:

```
[storage]
   path = /mnt/storage
   browseable = yes
   read only = no
   create mask = 0775
   directory mask = 0775
   valid users = mark
```

Create a Samba password for your user:

```sh
sudo smbpasswd -a mark
```

Restart Samba to apply changes:

```sh
sudo systemctl restart smbd
```

Verify Samba is running:

```sh
sudo systemctl status smbd
```

### Step 6: Access from Linux Clients

**Option A: Access via File Manager (GUI)**

Open your file manager and navigate to:

```
smb://192.168.1.167/storage
```

Enter username and Samba password when prompted.

**Option B: Mount Permanently via fstab**

Create a mount point on the client:

```sh
sudo mkdir -p /mnt/network-storage
```

Create a credentials file:

```sh
nano ~/.smbcreds
```

Add your credentials:

```
username=mark
password=YOUR_SAMBA_PASSWORD
```

Secure the credentials file:

```sh
chmod 600 ~/.smbcreds
```

Edit /etc/fstab:

```sh
sudo nano /etc/fstab
```

Add this line (replace 1000 with your actual UID/GID if different):

```
//192.168.1.167/storage /mnt/network-storage cifs credentials=/home/mark/.smbcreds,uid=1000,gid=1000 0 0
```

Reload systemd and mount:

```sh
sudo systemctl daemon-reload
sudo mount /mnt/network-storage
```

Verify the mount:

```sh
mount | grep network-storage
```

Test access:

```sh
ls /mnt/network-storage
```

### Step 7: Access from Windows Clients

**Option A: Map as Network Drive (GUI)**

1. Open File Explorer
2. Click "This PC" in the left sidebar
3. Click "Map network drive" in the toolbar
4. Choose a drive letter (e.g., Z:)
5. Enter the folder path: `\\192.168.1.167\storage`
6. Check "Reconnect at sign-in" if you want it permanent
7. Check "Connect using different credentials"
8. Click "Finish"
9. Enter username: `mark` and your Samba password
10. Click "OK"

**Option B: Access via Command Line**

Open Command Prompt or PowerShell:

```cmd
net use Z: \\192.168.1.167\storage /user:mark YOUR_SAMBA_PASSWORD
```

To make it persistent across reboots:

```cmd
net use Z: \\192.168.1.167\storage /user:mark YOUR_SAMBA_PASSWORD /persistent:yes
```

To disconnect:

```cmd
net use Z: /delete
```

### Verification

The network storage should now be accessible from:

- **Proxmox VM**: `/mnt/storage`
- **Linux clients**: `/mnt/network-storage` (if mounted) or `smb://192.168.1.167/storage` (file manager)
- **Windows clients**: `Z:\` (if mapped) or `\\192.168.1.167\storage` (UNC path)

All devices can read and write files to the shared storage location.
