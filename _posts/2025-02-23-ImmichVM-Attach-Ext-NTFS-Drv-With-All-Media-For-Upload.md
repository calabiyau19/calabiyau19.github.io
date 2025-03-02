---
layout: post
title: "Immich - Add External Drive via Samba"
draft: false
date:  2025-02-24
---





### **Project Documentation: Sharing an External NTFS Drive from Proxmox to an IMMICH VM via Samba**

#### **Objective:**

To mount an external **NTFS hard drive** (previously attached to a Windows 10 PC) on a **Proxmox server**, share it via **Samba**, and make it accessible to an **IMMICH VM**, allowing for fast and efficient photo uploads.

**Step-by-Step Guide**

### **🟢 Step 1: Identify & Verify the External Drive on Proxmox**

1️⃣ **List all available drives to identify the external one:**

bash  
Copy  
`lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,UUID`

✅ **Expected output:** The external drive (e.g., `sdc1`, NTFS formatted) should be listed, with a UUID assigned.  
✅ **In our case:** `sdc1` (NTFS, UUID: `F474B7AA74B76DCC`) was identified.

### **🟢 Step 2: Mount the External NTFS Drive on Proxmox**

1️⃣ **Check if the drive is already mounted:**

bash  
Copy  
`mount | grep sdc1`

✅ **If no output appears, manually mount it:**

bash  
Copy  
`mount -t ntfs-3g /dev/sdc1 /mnt/external_drive_k`

2️⃣ **Confirm the mount was successful:**

bash  
Copy  
`mount | grep sdc1`

✅ **Expected output:** `/dev/sdc1 on /mnt/external_drive_k type fuseblk (rw,...)`

### **🟢 Step 3: Ensure the Drive Mounts at Boot**

1️⃣ **Edit `/etc/fstab` to automatically mount the drive on reboot:**

bash  
Copy  
`nano /etc/fstab`

2️⃣ **Add this line at the bottom:**

ini  
Copy  
`UUID=F474B7AA74B76DCC /mnt/external_drive_k ntfs-3g defaults,uid=1000,gid=1000,dmask=027,fmask=137 0 0`

✅ **Why?**

* Ensures **read/write access** for Samba.  
* **Prevents permission issues** when accessing files.

3️⃣ **Test the `/etc/fstab` entry:**

bash  
Copy  
`mount -a`

✅ **No errors?** The drive will now auto-mount on reboot.

### **🟢 Step 4: Install & Configure Samba on Proxmox**

1️⃣ **Install Samba (if not installed):**

bash  
Copy  
`apt update && apt install -y samba`

2️⃣ **Backup the Samba config (optional but recommended):**

bash  
Copy  
`cp /etc/samba/smb.conf /etc/samba/smb.conf.bak`

3️⃣ **Edit the Samba config file:**

bash  
Copy  
`nano /etc/samba/smb.conf`

4️⃣ **Add this section at the bottom:**

ini  
Copy  
`[External_Drive_K]`  
   `path = /mnt/external_drive_k`  
   `read only = no`  
   `browsable = yes`  
   `guest ok = yes`  
   `force user = root`  
   `create mask = 0777`  
   `directory mask = 0777`

5️⃣ **Restart Samba to apply changes:**

bash  
Copy  
`systemctl restart smbd`

6️⃣ **Confirm Samba is running correctly:**

bash  
Copy  
`systemctl status smbd`

✅ **Expected output:** `"active (running)"`

### **🟢 Step 5: Verify the Samba Share from the IMMICH VM**

1️⃣ **On the IMMICH VM (192.168.1.141), install `smbclient` (if not already installed):**

bash  
Copy  
`sudo apt update && sudo apt install -y smbclient`

2️⃣ **List available shares on the Proxmox Samba server:**

bash  
Copy  
`smbclient -L //192.168.1.125 -N`

✅ **Expected output:**

`Sharename       Type`  
`---------       ----`  
`External_Drive_K Disk`

3️⃣ **Test access to the shared folder:**

bash  
Copy  
`smbclient //192.168.1.125/External_Drive_K -N`

✅ **Expected result:** An `smb: \>` prompt appears, allowing you to list files with `ls`.

### **🟢 Step 6: Access the Share Inside IMMICH & Upload Files**

1️⃣ **Open IMMICH's web interface** (`http://192.168.1.141`).  
2️⃣ **Go to the Upload section** and check for **network locations**:

* If **External\_Drive\_K doesn’t appear automatically**, go to **Other Locations**.  
* Select `root on 192.168.1.125`.  
* Navigate to `/mnt/external_drive_k` and verify that files are visible.  
  3️⃣ **Test Uploading Photos\!** 🎉

✅ **Expected outcome:** IMMICH successfully browses and uploads photos from `External_Drive_K`.

## **Troubleshooting & Fixes**

### **❌ Problem: "The folder contents could not be displayed" in IMMICH**

✅ **Fix:** Manually navigate to **Other Locations → root on 192.168.1.125 → mnt → external\_drive\_k**

### **❌ Problem: Old Windows network shortcut still appears**

✅ **Fix:** Right-click and **remove the old shortcut** from Nemo or the file manager cache:

bash  
Copy  
`rm -rf ~/.cache/gvfs`  
`nautilus -q`

**Final Outcome**

🎉 IMMICH now **sees and uploads photos from the NTFS external drive**, just like it did when it was connected to the Windows 10 machine. **Wireless bottlenecks are eliminated, and transfers are much faster.**

### **📝 Summary of Key Commands**

| Task | Command |
| ----- | ----- |
| List drives & identify NTFS disk | `lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,UUID` |
| Check if the drive is mounted | \`mount   \< this is incorrect |
| Manually mount NTFS drive | `mount -t ntfs-3g /dev/sdc1 /mnt/external_drive_k` |
| Auto-mount drive at boot | Edit `/etc/fstab` & add: `UUID=F474B7AA74B76DCC /mnt/external_drive_k ntfs-3g defaults,uid=1000,gid=1000,dmask=027,fmask=137 0 0` |
| Test entry | `/etc/fstabmount -a` |
| Install Samba | `apt update && apt install -y samba` |
| Restart Samba | `systemctl restart smbd` |
| Check Samba status | `systemctl status smbd` |
| List available Samba shares | `smbclient -L //192.168.1.125 -N` |
| Access the share via Samba | `smbclient //192.168.1.125/External_Drive_K -N` |
| Install `smbclient` on IMMICH VM | `sudo apt update && sudo apt install -y smbclient` |
| Remove old network cache (optional) | `rm -rf ~/.cache/gvfs && nautilus -q` |

**Final Thoughts**

This guide provides a **straightforward, step-by-step solution** to mounting, sharing, and accessing an **NTFS external hard drive** on **Proxmox**, allowing **IMMICH** to upload photos **exactly like it did when the drive was attached to Windows 10**.

