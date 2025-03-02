---
layout: post
title: "Immich - Ext. Storage For Processed Media"
draft: false
date:  2025-02-24
---


# **Summary: Setting Up External Storage for Immich on `nvme2tb-vm`**

### **🔹 Goal:**

* Attach a **400GB virtual disk** from `nvme2tb-vm` to `immich-vm` (Ubuntu VM).  
* Mount it at **`/mnt/immich-storage`** for Immich to store media.  
* Ensure **permissions and ownership** are correct so Immich can write to it.  
* Modify Immich’s **`.env` file** to use this storage.  
* Troubleshoot **`immich_server` restarting issues** related to missing `.immich` files.

## **Part 1: Adding a 400GB Virtual Disk in Proxmox**

1. **Opened the Proxmox Web UI.**  
2. **Added a new 400GB virtual disk** to `immich-vm`:  
   * **Storage:** `nvme2tb-vm`  
   * **Format:** `QCOW2` (so snapshots can be used)  
   * **Bus/Device:** `VirtIO`  
3. **Rebooted `immich-vm` to apply changes.**

## **Part 2: Formatting & Mounting the Disk Inside Ubuntu (`immich-vm`)**

### **1️⃣ Verify the New Disk is Detected**

bash  
   
`lsblk`

✅ **Confirmed `/dev/sdb` was the new 400GB disk.**

### **2️⃣ Check If the Disk Has a Filesystem**

bash  
   
`sudo blkid /dev/sdb`

❌ **No output → Disk was unformatted.**

✅ **Next Step: Format as Ext4.**

### **3️⃣ Format the Disk as Ext4**

bash  
   
`sudo mkfs.ext4 /dev/sdb`

✅ **Disk was successfully formatted.**

### **4️⃣ Create a Mount Point for the Disk**

bash  
   
`sudo mkdir -p /mnt/immich-storage`

✅ **Mount point created at `/mnt/immich-storage`.**

### **5️⃣ Mount the Disk Temporarily**

bash  
   
`sudo mount /dev/sdb /mnt/immich-storage`

✅ **Confirmed the disk was mounted.**

### **6️⃣ Verify the Disk is Mounted**

bash  
   
`df -h | grep sdb`

✅ **Confirmed `/mnt/immich-storage` was using 400GB.**

## **Part 3: Making the Mount Permanent (Survives Reboots)**

### **1️⃣ Get the Disk’s UUID**

bash  
   
`sudo blkid /dev/sdb`

✅ **Output showed:**

ini  
   
`UUID="43d69a9b-d15a-48b1-a703-395a04fd77af"`

### **2️⃣ Add the Disk to `/etc/fstab`**

bash  
   
`sudo nano /etc/fstab`

✅ **Added this line at the bottom (using the actual UUID):**

ini  
   
`UUID=43d69a9b-d15a-48b1-a703-395a04fd77af /mnt/immich-storage ext4 defaults 0 2`

✅ **Saved and exited (`CTRL + X`, `Y`, `Enter`).**

### **3️⃣ Test the fstab Entry**

bash  
   
`sudo mount -a`

✅ **No errors → Mount will persist after reboot.**

## **Part 4: Fixing Ownership & Permissions**

### **1️⃣ Check Who Owns `/mnt/immich-storage`**

bash  
   
`ls -ld /mnt/immich-storage`

❌ **It was owned by `root root` → Immich wouldn’t be able to write to it.**

### **2️⃣ Change Ownership to `mark`**

bash  
   
`sudo chown -R mark:mark /mnt/immich-storage`

✅ **Now owned by `mark mark`.**

### **3️⃣ Set Correct Permissions**

bash  
   
`sudo chmod -R 775 /mnt/immich-storage`

✅ **Ensured Immich can read/write files.**

## **Part 5: Telling Immich to Use `/mnt/immich-storage`**

### **1️⃣ Open the Immich `.env` File**

bash  
   
`nano ~/immich/docker/.env`

✅ **Changed this line:**

ini  
   
`UPLOAD_LOCATION=./library`

👉 **To this:**

ini  
   
`UPLOAD_LOCATION=/mnt/immich-storage`

✅ **Saved and exited (`CTRL + X`, `Y`, `Enter`).**

### **2️⃣ Restart Immich**

bash  
   
`cd ~/immich/docker`  
`docker compose down`  
`docker compose up -d`

🚨 **Problem: `immich_server` kept restarting.**

## **Part 6: Troubleshooting `immich_server` Restarting**

### **1️⃣ Checked Logs for Errors**

bash  
   
`docker logs --tail=50 immich_server`

🚨 **Error:**

pgsql  
   
`Failed to read upload/encoded-video/.immich: ENOENT: no such file or directory`

**Cause:** Immich **expected certain folders to exist**, but they were missing.

### **2️⃣ Manually Create the Required Folders**

bash  
   
`sudo mkdir -p /mnt/immich-storage/upload \`  
               `/mnt/immich-storage/library \`  
               `/mnt/immich-storage/thumbs \`  
               `/mnt/immich-storage/encoded-video \`  
               `/mnt/immich-storage/profile \`  
               `/mnt/immich-storage/backups`

✅ **Created all required folders.**

### **3️⃣ Verify Folder Creation**

bash  
   
`ls -l /mnt/immich-storage`

✅ **Confirmed all folders existed.**

---

### **4️⃣ Manually Create `.immich` Files in Each Folder**

bash  
   
`touch /mnt/immich-storage/upload/.immich \`  
      `/mnt/immich-storage/library/.immich \`  
      `/mnt/immich-storage/thumbs/.immich \`  
      `/mnt/immich-storage/encoded-video/.immich \`  
      `/mnt/immich-storage/profile/.immich \`  
      `/mnt/immich-storage/backups/.immich`

✅ **Created empty `.immich` files inside each folder.**

### **5️⃣ Verify `.immich` Files Exist**

bash  
   
`ls -la /mnt/immich-storage/* | grep .immich`

✅ **Confirmed all `.immich` files were present.**

### **6️⃣ Restart Immich Again**

bash  
   
`cd ~/immich/docker`  
`docker compose down`  
`docker compose up -d`

✅ **`immich_server` now runs without crashing.** 🎉

# **Summary of Issues & Fixes**

| Issue | Fix |
| ----- | ----- |
| Disk was not formatted | Used `mkfs.ext4` to format it |
| Disk did not mount at boot | Used `UUID` in `/etc/fstab` |
| Immich could not write to storage | Fixed ownership (`chown mark:mark`) and permissions (`chmod 775`) |
| `immich_server` kept restarting | Created missing storage folders & `.immich` marker files |

# **Final Status**

* **Storage is mounted at `/mnt/immich-storage`**  
* **Immich is storing media in the new location**  
* **The server is running without restarts**

**Phase Two was successfully completed\!**

