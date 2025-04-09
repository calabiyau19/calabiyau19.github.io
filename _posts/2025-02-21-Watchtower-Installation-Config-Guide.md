---
layout: post
title: "Watchtower Install"
draft: false
date:  2025-02-21
description:  Step by step directions to install, setup and run Watchtower which will run inside my VMs that have Docker containers (all of them!) and update them automatically.  This is the initial version of watchtower (containerrr) and an updated version (nickfedor)
---

## UPDATE: 04192025

The containrrr/watchtower version of watchtower is no longer actively maintained.  There are currently several forks being maintained so I went with the nickfedor/watchtower version from github.  At the end of this post are the steps I followed to change each instance of watchtower running in the home lab now. Six VMs on two hypervisors running 22 containers and one watchtower instance on lpt-hp running 14 containers.

Watchtower Installation & Configuration Guide

###  What This Does

This setup ensures:   **Automatic updates** for all running Docker containers.  
  **Ntfy.sh notifications** whenever a container is updated.  
  **Customizable update check interval** (default: every **24 hours**).

---

###  Step 1: Install Watchtower

1️⃣ **SSH into the VM where Watchtower will run**

-bash-  
   
`ssh vm-user@vm-ip`    (e.g. ssh mark@192.168.1.108)

2️⃣ **Pull the latest Watchtower image**

-bash-  
   
`docker pull containrrr/watchtower`

3️⃣ **Run Watchtower with notifications & a custom interval**

-bash-  
   
`docker run -d \`  
  `--name watchtower \`  
  `--restart unless-stopped \`  
  `-v /var/run/docker.sock:/var/run/docker.sock \`  
  `-e WATCHTOWER_NOTIFICATIONS=shoutrrr \`  
  `-e WATCHTOWER_NOTIFICATION_URL="ntfy://ntfy.sh/watchtower-updates" \`  
  `containrrr/watchtower --interval 43200`

 **What this does:**

* `-d` → Runs in detached mode (background).  
* `--restart unless-stopped` → Ensures Watchtower restarts if the VM reboots.  
* `-v /var/run/docker.sock:/var/run/docker.sock` → Allows Watchtower to monitor containers.  
* `-e WATCHTOWER_NOTIFICATIONS=shoutrrr` → Enables notifications via **ntfy.sh**.  
* `-e WATCHTOWER_NOTIFICATION_URL="ntfy://ntfy.sh/watchtower-updates"` → Sends alerts to **ntfy.sh**.  
* `--interval 43200` → **Check every 12 hours** (**adjust as needed**).

---

###  Step 2: Verify That Watchtower is Running

Check if Watchtower is running and monitoring containers:

-bash-  
   
`docker logs watchtower | grep "Scheduling first run"`

  If everything is correct, you should see something like:

-bash-  
   
`time="2025-02-22T23:02:32Z" level=info msg="Scheduling first run: 2025-02-23 11:02:32 +0000 UTC"`  
`time="2025-02-22T23:02:32Z" level=info msg="Next check in 11 hours, 59 minutes, 59 seconds"`

---

###  Step 3: Test Ntfy.sh Notifications

To make sure notifications work, **force an update check**:

-bash-  
   
`docker exec watchtower /watchtower --run-once`

Then, open [**https://ntfy.sh/watchtower-updates**](https://ntfy.sh/watchtower-updates) in a browser (or check your **ntfy mobile app**) to confirm you received an update notification.

---

###  Step 4: Customize Watchtower (Optional)

### ** Adjust Update Check Interval**

Change the `--interval` value when starting Watchtower:

* **Every 6 hours:** `--interval 21600`  
* **Every 24 hours:** `--interval 86400`  
* **Every 3 days:** `--interval 259200`

To change the interval, stop and remove Watchtower:

-bash-  
   
`docker stop watchtower`  
`docker rm watchtower`

Then re-run the `docker run` command with the new interval.

---

### **Update Only Specific Containers**

By default, Watchtower updates **all containers**. To limit updates to specific containers, add container names at the end of the command:

-bash-  
   
`docker run -d \`  
  `--name watchtower \`  
  `--restart unless-stopped \`  
  `-v /var/run/docker.sock:/var/run/docker.sock \`  
  `-e WATCHTOWER_NOTIFICATIONS=shoutrrr \`  
  `-e WATCHTOWER_NOTIFICATION_URL="ntfy://ntfy.sh/watchtower-updates" \`  
  `containrrr/watchtower mealie portainer`

**This will only update `mealie` and `portainer`, leaving other containers untouched.**

---

###   Step 5: Final Checklist

Before calling it **done**, make sure:  
  Watchtower is running → `docker ps | grep watchtower`  
  Notifications work → Check [**https://ntfy.sh/watchtower-updates**](https://ntfy.sh/watchtower-updates)  
  Update interval is correct → `docker logs watchtower | grep "Scheduling first run"`

---

### TL;DR \- Quick Commands

-bash-  
   
`ssh vm-user@vm-ip`  
`docker pull containrrr/watchtower`  
`docker run -d \`  
  `--name watchtower \`  
  `--restart unless-stopped \`  
  `-v /var/run/docker.sock:/var/run/docker.sock \`  
  `-e WATCHTOWER_NOTIFICATIONS=shoutrrr \`  
  `-e WATCHTOWER_NOTIFICATION_URL="ntfy://ntfy.sh/watchtower-updates" \`  
  `containrrr/watchtower --interval 43200`  
`docker logs watchtower | grep "Scheduling first run"`  
`docker exec watchtower /watchtower --run-once  # Test notifications`

---

## UPDATE: 04192025

How to change watchtower version to nickfedor/watchtower

{% raw %}
```sh
docker ps --format "table {{.Names}}\t{{.Image}}"
```
{% endraw %}

This will get the actual name of the watchtower container running.  Mine were being given random names as watchtower was trying to update them and failing and then giving them a random string name.  If needed run the following command.

```sh
docker rename [container-name-from-prior-command] watchtower
```

Then run:

```sh
docker stop watchtower && docker rm watchtower
```

Now reinstall and restart the Watchtower container

```sh

docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATIONS=shoutrrr \
  -e WATCHTOWER_NOTIFICATION_URL="ntfy://ntfy.sh/watchtower-updates" \
  -e WATCHTOWER_NOTIFICATION_REPORT=true \
  nickfedor/watchtower \
  --interval 172800 \
  --cleanup \
  --include-restarting \
  --label-enable\

```
NOTE:  This command turns off Watchtower trying to update itself. I will update that manually if needed - but that will be infrequently going forward.  


Wait a minute or so and then run the following to confirm it is running and healthy:

```sh
docker ps
```

