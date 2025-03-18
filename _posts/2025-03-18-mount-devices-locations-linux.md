---
layout: post
title: "Mount Devices and Locations in Linux"
date:  2025-03-18
---

## NOTE: run these commands individually 

### Attach an External USB Drive (HDD/SSD)

```sh  
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT
sudo mkdir -p /mnt/external
sudo mount /dev/sdX1 /mnt/external
sudo chown -R $USER:$USER /mnt/external
sudo chmod -R 777 /mnt/external
blkid /dev/sdX1
sudo nano /etc/fstab
# Add: UUID=YOUR-UUID /mnt/external ext4 defaults,nofail,x-systemd.device-timeout=10 0 2
sudo mount -a
```

### Attach a Windows Network Share (SMB)
  
```sh  
sudo apt update && sudo apt install cifs-utils -y
sudo mkdir -p /mnt/networkdrive
sudo mount -t cifs //192.168.1.X/SharedFolder /mnt/networkdrive -o username=User,password=Pass,vers=3.0
sudo nano /etc/fstab
# Add: //192.168.1.X/SharedFolder /mnt/networkdrive cifs username=User,password=Pass,vers=3.0,nofail,x-systemd.automount 0 0
sudo mount -a
```

### Attach a Linux Network Share (NFS)

```sh
sudo apt update && sudo apt install nfs-common -y
sudo mkdir -p /mnt/networkdrive
sudo mount -t nfs 192.168.1.X:/home/shared /mnt/networkdrive
sudo nano /etc/fstab
# Add: 192.168.1.X:/home/shared /mnt/networkdrive nfs defaults,nofail,x-systemd.automount 0 0
sudo mount -a
```

### Attach an Internal Drive or Partition

```sh
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT
sudo mkfs.ext4 /dev/nvme0n1p3  # (If not formatted)
sudo mkdir -p /mnt/internal
sudo mount /dev/nvme0n1p3 /mnt/internal
sudo chown -R $USER:$USER /mnt/internal
sudo chmod -R 777 /mnt/internal
blkid /dev/nvme0n1p3
sudo nano /etc/fstab
# Add: UUID=YOUR-UUID /mnt/internal ext4 defaults,nofail,x-systemd.device-timeout=10 0 2
sudo mount -a
```

### Attach a Network Location in Nemo (Linux Mint)
```sh
nemo smb://192.168.1.X  # For Windows share
nemo nfs://192.168.1.X  # For Linux NFS share
```
