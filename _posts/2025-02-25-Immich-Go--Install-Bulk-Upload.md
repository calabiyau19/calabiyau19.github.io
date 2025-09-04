---
layout: post
title: "Immich Go Bulk Uploader - Installation and Use"
draft: false
date:  2025-02-25
---


# [**Link**](https://chatgpt.com/g/g-AaCO9ep8Y-linux-specialist/c/67bcebdd-2844-8013-b699-52db568941b1) **to full Installation**

# **Installing & Using Immich-Go for Bulk Photo Uploads**

## **Overview**

Immich-Go allows bulk photo uploads to an Immich server while preserving folder structure as albums. This guide covers installation, setup, testing, troubleshooting, and best practices.

---

## **Prerequisites**

Ensure you have:

* SSH access to the Proxmox server where the USB HDD is attached. 192.168.1.125  
* A running Immich server (e.g., `192.168.1.141`).  
* An API key from Immich for authentication.(Under User \- not Admin)

---

## **Step 1: Generate an API Key in Immich**

1. Log into the Immich Web UI at `http://192.168.1.141:2283`.  
2. Navigate to **Account Settings → API Keys**.  
3. Click **"New API Key"**, name it `immich-go`, and copy the key.

---

## **Step 2: Install Immich-Go on Proxmox**

SSH into Proxmox:  
bash  
   
`ssh root@<proxmox-ip>`

1. Replace `<proxmox-ip>` with the actual IP of your Proxmox server.

Download and install Immich-Go:  
bash  
   
`cd /usr/local/bin`  
`wget https://github.com/simulot/immich-go/releases/download/v0.24.2/immich-go_Linux_x86_64.tar.gz`  
`tar -xzf immich-go_Linux_x86_64.tar.gz`  
`rm immich-go_Linux_x86_64.tar.gz`  
`chmod +x immich-go`

2. 

Verify installation:  
bash  
   
`immich-go --version`

3. Expected output: `immich-go version 0.24.2`

---

## **Step 3: Find USB HDD Mount Path**

List mounted drives:  
bash  
   
`lsblk -o NAME,MOUNTPOINT,SIZE,FSTYPE`  
Example output:  
r  
   
`sdc                                                       1.8T`   
`└─sdc1                            /mnt/external_drive_k   1.8T ntfs`  
This confirms the USB HDD is mounted at:  
bash  
   
`/mnt/external_drive_k`

1. 

---

## **Step 4: Perform a Test Upload**

Upload a small test folder:  
bash  
   
`immich-go upload from-folder --server=http://192.168.1.141:2283 --api-key=<your-api-key> "/mnt/external_drive_k/Christmas/Christmas 2004"`

1.   
2. Verify in the Immich Web UI:  
   * Log into `http://192.168.1.141:2283`.  
   * Check the **Library** to confirm the photos are visible.  
   * Check **Albums** to confirm the folder name appears as an album.

---

## **Step 5: Bulk Upload While Keeping Folder Structure as Albums**

Upload all photos while maintaining folder names as albums:  
bash  
   
`immich-go upload from-folder --server=http://192.168.1.141:2283 --api-key=<your-api-key> --folder-as-album=FOLDER "/mnt/external_drive_k/ALL_PICTURES_FromBackupFiles"`

1. Each **subfolder** becomes an album (e.g., "Xmas 2017", "Xmas 2018"), and the main folder's photos go into an album with the same name.

---

## **Step 6: Enable Tracking & Prevent Duplicates**

Use `--session-tag` to track uploads:  
bash  
   
`immich-go upload from-folder --server=http://192.168.1.141:2283 --api-key=<your-api-key> --folder-as-album=FOLDER --session-tag "/mnt/external_drive_k/ALL_PICTURES_FromBackupFiles"`

1. This helps track uploads and makes it easier to remove or review them if needed.

---

## **Step 7: Handling Duplicates**

1. Check for duplicates in the Immich Web UI:  
   * Navigate to **Library → Duplicates**.  
   * Review and remove as needed.

Run a dry run before large uploads to preview what will be uploaded:  
bash  
   
`immich-go upload from-folder --server=http://192.168.1.141:2283 --api-key=<your-api-key> --folder-as-album=FOLDER --dry-run "/mnt/external_drive_k/ALL_PICTURES_FromBackupFiles"`

2. This will show the files that would be uploaded without actually uploading them.

---

## **Key Takeaways**

* Folder structure is preserved using `--folder-as-album=FOLDER`.  
* The `--session-tag` option helps track and manage uploads.  
* Running `--dry-run` before large uploads prevents mistakes.  
* Checking "Library → Duplicates" in Immich helps remove duplicate files.

This completes the Immich-Go bulk upload setup.

