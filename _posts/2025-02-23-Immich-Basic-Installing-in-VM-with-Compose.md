---
layout: post
title: "Immich Default Install using Docker in a VM"
draft: false
date:  2025-02-21
---




###  **Phase I: Installing Immich on Ubuntu (Before Setting Up External Storage)**

#### **🔹 Goal:**

* Install **Immich** on an **Ubuntu 24.04 VM** running on **Proxmox**.  
* Ensure **Docker and Docker Compose** are installed and working.  
* Run Immich using **Docker Compose** with its default settings.  
* Verify that the installation is successful by uploading and viewing a test photo.

---

###  **Step 1: Prepare the Ubuntu VM**

#### **1️⃣ Verify Ubuntu Version**

bash  
   
`lsb_release -a`

✅ **Expected Output:**

   
`Distributor ID: Ubuntu`  
`Description:    Ubuntu 24.04.2 LTS`  
`Release:        24.04`  
`Codename:       noble`

 **Why?** Ensures we are working on the correct Ubuntu version.

git \--version  
curl \--version  
jq \--version

**ME \- they are all installed.**


#### **2️⃣ Verify That Docker & Docker Compose Are Installed**

bash  
   
`docker --version`  
`docker compose version`

✅ **Expected Output (Example):**

`Docker version 28.0.0, build f9ced58`  
`Docker Compose version v2.33.0`

**Why?** Confirms that **Docker & Docker Compose** are installed correctly.

#### **Step 2: Download Immich**

#### **1️⃣ Clone the Immich Repository**

bash  
   
`git clone https://github.com/immich-app/immich.git`  
`cd immich`

✅ **What this does:**

* Downloads the **latest version of Immich** from GitHub.  
* **Moves into the Immich directory.**

***THE BASE DIRECTORY FOR IMMICH***   
***\- WHERE DOCKER COMPOSE UP/DOWN WILL BE RUN***   
***\-AND THE .ENV FILE IS AS WELL IS :***  
***/HOME/MARK/IMMICH/DOCKER***

**This is different than in previous installations on Immich we have done.  It is a different location because we cloned the git repository and this method made the Immich folder by default to match the git project name since one was not specified.  And in the Immich folder is the docker folder by default**

#### **How the Folder Structure Was Created** **After cloning, your folder structure looked like this:**

**/home/mark/**

**│── immich/               \<-- Created by \`git clone\`|**  
	**│   │── docker/           \<-- Contains \`docker-compose.yml\` & \`.env\`**  
	**│   │── server/**  
	**│   │── web/**  
	**│   │── mobile/**  
	**│   │── README.md**

**This meant:**

* **All Docker setup files were inside /home/mark/immich/docker/.**  
* **That’s why we needed to run all Docker commands (docker compose up \-d, editing .env, etc.) from inside the docker/ folder.**

#### **2️⃣ Check Out the Latest Stable Release**

Since `main` can have unstable changes, we switch to the latest **stable release**.

##### **Find the latest release:**

bash  
   
`git tag | grep -E '^v[0-9]+' | sort -V | tail -n1`

✅ **Example Output:**

 `v1.126.1`

##### **Switch to the latest release:**

bash  
   
`git checkout v1.126.1`

`git status     confirms switch to 1.126.1`

✅ **Why?** Ensures we use a **stable, tested version** instead of potentially unstable development code.

###  **Step 3: Configure Environment Variables**

#### **1️⃣ Copy the Default `.env` File**

bash  
   
`cp docker/example.env docker/.env`

✅ **What this does:**

* Creates a `.env` file that Immich will use for configuration.

#### **2️⃣ Open and Edit the `.env` File**

`nano docker/.env`

##### **Check These Key Settings:**

* **Database settings** (Leave as default; Docker will handle this).  
  (I changed the DB password here, so it is not the default \- it is always a good practice)  
* **Storage Location** (`UPLOAD_LOCATION` defaults to `./library` for now).  (default upload “storage” location)  
* **Port Configuration** (Default: `IMMICH_SERVER_PORT=2283`).

**Timezone (Optional, set to match your location)**  
   
`TZ=America/New_York`

✅ **Save and exit:**

* Press **CTRL \+ X**, then **Y**, then **Enter**.

**NOTE:  No changes were made to docker-compose.yml since everything was left on the default settings and the .env file filled in a few variables**

### **Make sure docker is running**:

### `docker info` 

###  **Step 4: Start Immich Using Docker Compose**

#### **1️⃣ Move into the Docker Directory**

bash  
 `cd docker`

#### **2️⃣ Start Immich Containers**

bash  
   
`docker compose up -d`

✅ **What this does:**

* Pulls and starts **all Immich containers** (server, database, Redis, machine learning).  
* Runs them in the **background (`-d`)**.

#### **3️⃣ Verify That All Containers Are Running**

bash  
   
`docker ps -a`

✅ **Expected Output (Example):**

`CONTAINER ID   IMAGE                                       STATUS`  
`a1b2c3d4e5f6   ghcr.io/immich-app/immich-server:release   Up`  
`b2c3d4e5f6g7   ghcr.io/immich-app/immich-machine-learning Up`  
`c3d4e5f6g7h8   tensorchord/pgvecto-rs:pg14-v0.2.0         Up`  
`d4e5f6g7h8i9   redis:6.2-alpine                           Up`

💡 **If all containers are "Up," Immich is running correctly\!**

#### **1️⃣ Open Immich in a Web Browser**

[`http://immich-vm:2283`](http://immich-vm:2283) `- or http://immich-vm-ip:2283`

✅ **Expected Result:** You should see the **Admin Registration page**.

#### **2️⃣ Register the Admin Account**

* **Username:** `admin`  
* **Password:** (Set a secure password)  
* **Email:** (Choose any admin email)  
* **First Name / Last Name:** (Optional)  
  ✅ **Click "Register."**

###  **Step 6: Upload a Test Photo**

#### **1️⃣ Log in to Immich**

* Use the **admin account** created in Step 5\.

#### **2️⃣ Upload a Small Test Image**

* Click **"Upload"**  
* Select a test photo from your computer  
* Verify that it **appears in Immich**

#### **3️⃣ Verify That the File Was Saved to Disk**

bash  
   
`ls -lah ~/immich/docker/library/upload/`

✅ **If the file appears here, Immich is working\!** 🎉

---

###  **Problems Encountered & Fixes**

| Problem | Fix |
| ----- | ----- |
| Immich containers **not starting** | Checked logs with `docker logs --tail=50 immich_server`, found `.env` misconfiguration, fixed settings. |
| Web interface **not loading** | Checked if server was running (`docker ps`), found missing `.env` file, copied it. |
| Database connection issues | Restarted containers with `docker compose down && docker compose up -d`. |
| Uploads not working | **Storage permissions issue** (fixed in Phase II by setting correct ownership). |

---

### **✅ Phase I Completed\!**

🚀 **Immich was successfully installed and tested with a basic upload.**  
📌 **At this point, Immich was still using the default upload location (`./library/upload/`).**  
⏭️ **Phase II began when we switched to external storage (`/mnt/immich-storage`).**

