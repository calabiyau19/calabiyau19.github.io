---
layout: post
title: "Docker Compose Install on Ubuntu server 24.04"
date:  2025-03-31
description:  Step by step installation of Docker and Docker Compose in a Ubuntu 24.04 Virtual Machine running on Proxmox server
---

### **Installing the Latest Docker and Docker Compose on Ubuntu Server 24.04**

This guide ensures you install the latest **Docker Engine** and **Docker Compose** on **Ubuntu Server 24.04 (Noble)**, using the official Docker repositories.


## **Step 1: Update the System**

Ensure your system is up to date before installing any packages:

```sh
sudo apt update && sudo apt upgrade -y
```


## **Step 2: Install Required Dependencies**

Install required system packages:

```sh
sudo apt install -y ca-certificates curl gnupg
```


## **Step 3: Add Docker’s Official GPG Key**

Docker packages are signed, so we need to add the official GPG key:

```sh
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc

```


## **Step 4: Add the Docker Repository (For Ubuntu 24.04 Noble)**

Now, add the **Docker stable repository** to your system:

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


Then update the package index again:

```sh
sudo apt update
```


## **Step 5: Install Docker Engine and Docker Compose Plugin**

Now, install the latest versions of Docker and the Compose plugin:

```sh
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## **Step 6: Enable and Start the Docker Service**

Ensure that Docker starts on boot and is currently running:

```sh
sudo systemctl enable --now docker
```


Check the service status:

```sh
sudo systemctl status docker
```

## **Step 7: Verify Installation**

Check that Docker is installed correctly:

```sh
docker --version
```

Verify the Docker Compose plugin:

```sh
docker compose version
```
Run a test container to ensure everything works:

```sh
docker run hello-world
```

## **Step 8: Add Your User to the Docker Group (Optional, for Non-root Access)**

By default, you need `sudo` to run Docker commands. To run Docker as a regular user, add yourself to the **docker** group:

```sh
sudo usermod -aG docker $USER
```

# IMPORTANT: Log out and back in to apply group membership
# Do NOT use 'newgrp docker' — it temporarily changes your primary group and may cause permission issues

Now test running Docker **without sudo**:

```sh
docker run hello-world
```

## **Step 9: Keeping Docker and Compose Updated**

Since Docker releases frequent updates, update your system regularly:

```sh
sudo apt update && sudo apt upgrade -y
```

For a more targeted update, check if a new Docker version is available:

```sh
apt list --upgradable | grep docker
```

Then update Docker only:

```sh
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### **Final Verification**

After completing the steps, verify the installation one last time:

```sh
docker --version
docker compose version  
docker run hello-world
```

At this point, you have **the latest Docker and Docker Compose installed on Ubuntu 24.04**, fully functional and ready to use.

###### 

### **Final Step: Install Watchtower for Automatic Container Updates AFTER application installation**

Watchtower automatically updates your running Docker containers when new versions are available. However, **you should install Watchtower after deploying key applications like FreshRSS** to ensure everything is set up correctly before automating updates.


### **Step 11: Install Watchtower with Ntfy.sh Notifications**

**Run Watchtower with Ntfy.sh**  
Replace `<your-ntfy-topic>` with your chosen topic (e.g., `freshrss-updates`):

```sh
docker run -d
  --name watchtower  
  --restart unless-stopped
  -v /var/run/docker.sock:/var/run/docker.sock  
  -e WATCHTOWER_NOTIFICATIONS=ntfy
  -e WATCHTOWER_NOTIFICATION_URL="https://ntfy.sh/<your-ntfy-topic>" 
  containrrr/watchtower
```

1. 

**Verify Watchtower is Running**  
 Check logs to confirm Watchtower is monitoring containers and sending notifications:

```sh
`docker logs -f watchtower`
```

3. **Test Ntfy.sh Notifications**  
  Open your Ntfy.sh topic URL (`https://ntfy.sh/<your-ntfy-topic>`) in a browser or use the mobile app to verify Watchtower messages.

