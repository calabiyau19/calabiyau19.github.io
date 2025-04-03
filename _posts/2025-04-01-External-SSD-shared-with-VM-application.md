---
layout: post
title: "External SSD with VM application"
date:  2025-04-01
description:  Step by step directions of attaching a SSD to a Proxmox host and adding a folder to it that is accessible by a VM on the Proxmox server.  In this case, that was Navidrome on the Proxmox server with the music stored on the external ssd drive
---


 
 Here are the steps to properly set up the external SSD (e.g., `Ext_T7_ssd`) for sharing music from the Proxmox host to `navidrome-vm`.


###  **ON THE PROXMOX HOST**


### **1\. Plug in the External SSD (if not already done)**

Physically connect your `Ext_T7_ssd` drive via USB.


### **2\. Identify the SSD device and partitions**

Run:

```sh
lsblk -f  
```

This will show something like:

```sh  
NAME   FSTYPE LABEL       UUID                                 MOUNTPOINT  
sda
├─sda1 ext4   rootdisk    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /  
sdb
└─sdb1 ext4   Ext_T7_ssd  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  
```

Let’s say `sdb1` is your SSD partition.


### **3\. Mount the partition**

```sh
sudo mkdir -p /mnt/Ext_T7_ssd  
sudo mount /dev/sdb1 /mnt/Ext_T7_ssd  
```

Replace `/dev/sdb1` with the correct device name from lsblk.


### **4\. Make the mount permanent (optional but recommended)**

Edit /etc/fstab:

```sh
sudo nano /etc/fstab  
```

Add a line like:

```sh
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/Ext_T7_ssd ext4 defaults 0 2 
```

Replace the UUID with the actual one from `lsblk -f`.

Save and exit.


### **5\. Create the music folder**

```sh
sudo mkdir -p /mnt/Ext_T7_ssd/music  
```


### **6\. Set ownership or permissions (if needed)**

```sh
sudo chown -R mark:mark /mnt/Ext_T7_ssd/music
```


Now you’re **ready** to proceed to the NFS export steps

### **ON THE PROXMOX HOST**


### **1\. Mount the external SSD (if not already mounted)**

If Proxmox already sees the SSD (e.g. `/dev/sdX`), mount it like this:

```sh

sudo mkdir -p /mnt/Ext_T7_ssd

sudo mount /dev/sdX1 /mnt/Ext_T7_ssd

```

Replace `/dev/sdX1` with the correct partition (you can check with `lsblk`).


### **2\. Create the music folder on the SSD**

```sh

sudo mkdir -p /mnt/Ext_T7_ssd/music

```


### **3\. (Optional) Set permissions for Navidrome or a specific user**

```sh

sudo chown -R navidrome:navidrome /mnt/Ext_T7_ssd/music

```

Or, if using your user (e.g. `mark`):

```sh

sudo chown -R mark:mark /mnt/Ext_T7_ssd/music

```

Now, the folder is ready to store music files.


## **Corrected, Precise Summary (with terminal context)**


### **ON THE PROXMOX HOST**


#### **4\. Install NFS server (if not already installed):**

```sh

sudo apt update && sudo apt install nfs-kernel-server

```

---

#### **5\. Configure the export:**

```sh

sudo nano /etc/exports

```

Add this line:

```sh

/mnt/Ext_T7_ssd/music 192.168.1.0/24(ro,sync,no_subtree_check)
```


#### **6\. Reload NFS exports:**

```sh

sudo exportfs -ra

```


#### **7\. Verify NFS is exporting:**

```sh

sudo exportfs -v

```

You should see:

```sh

/mnt/Ext_T7_ssd/music  192.168.1.0/24(...)

```


### **ON THE `navidrome-vm` (Ubuntu 24.04)**


#### **8\. Install NFS client:**

```sh

sudo apt update && sudo apt install nfs-common

```


#### **9\. Create the mount point:**

```sh

sudo mkdir -p /mnt/navidrome/music

```

#### **10\. Mount the shared music folder:**

```sh

sudo mount -t nfs 192.168.1.125:/mnt/Ext_T7_ssd/music /mnt/navidrome/music


Verify:

```sh

ls -lah /mnt/navidrome/music

```

You should see your music folders.



#### **11\. Update `docker-compose.yml`:**

```sh

nano ~/navidrome/docker-compose.yml
```


Update `volumes:` section:

```sh

   volumes:

      - "${HOME}/navidrome/config:/data"

      - "/mnt/navidrome/music:/music:ro"

```

#### **12\. Restart Navidrome:**

```sh

cd ~/navidrome

docker compose down

docker compose up -d

```

#### **13\. Verify in browser:**

```sh

http://<navidrome-vm-ip>:4533



The full external music library should now be visible inside Navidrome.

