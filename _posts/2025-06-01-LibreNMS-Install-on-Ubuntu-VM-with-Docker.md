---
layout: post
title: "Setting up LibreNMS and SNMP clients on Network servers"
draft: false
date:  2025-06-01
description: How to set up LibreNMS and SNMP on each network server to monitor up/down, health and ports.  
---


#### **LibreNMS and SNMP Client Installation on Linux servers and VMs via Docker and custom scripts.** 

Prerequisites  
Ubuntu VM (librenms-vm) // Specs: 2 vCPUs, 4GB RAM, 32GB disk  
Docker and Docker Compose are already installed, and Hello World tested. 

Step 1: Clone the LibreNMS Docker Repository  
    

```
git clone https://github.com/librenms/docker.git librenms-docker  
cd \~/librenms-docker/examples/compose
```


This pulls the official LibreNMS Docker project and navigates to the example Docker Compose files.

Step 2: Configure the .env File  
   

```
nano .env
```


You customized environment variables like:

TZ=America/New\_York  
LIBRENMS\_SNMP\_COMMUNITY=public  
LIBRENMS\_LOCATION=Home Lab

Saved and exited.

Step 3: Deploy LibreNMS Using Docker Compose  
   

```
docker compose up \-d
```


Started all the LibreNMS services (MySQL, Redis, SNMP, dispatcher, etc.) in detached mode.

Check containers:


```
docker compose ps  
docker ps
```


Step 4: Access LibreNMS Web Interface  
Open a browser and go to:

[http://192.168.1.176:8000](http://192.168.1.176:8000)

You walk through the web installer to finish the setup steps.   No command-line steps here.

Step 5: Install SNMP Locally for Testing & Polling  
    

```
sudo apt install snmpd \-y  
sudo nano /etc/snmp/snmpd.conf
```


You set the SNMP community and location:  
   
rocommunity **public**  
sysLocation **Home Lab**  
sysContact admin@yourdomain.local  
agentAddress udp:161

Restart and enable SNMP:  
   

```
sudo systemctl restart snmpd  
sudo systemctl enable snmpd
```


Verify SNMP is running:  
   
```
sudo systemctl status snmpd  
sudo ss \-lnup | grep 161

```
Step 6: Test SNMP from Host and Inside Container  
From the host:  
   

```
snmpwalk \-v2c \-c public localhost system
```


If missing:  
   
```
sudo apt install snmp snmp-mibs-downloader \-y
```


From inside the LibreNMS container:  
   
```
docker exec \-it librenms    
snmpwalk \-v2c \-c public 192.168.1.125 system
```


Repeated for other devices:

192.168.1.100 (lpt-hp)  
192.168.1.108, 1.121, 1.141, etc.

Step 7: Check Storage Usage of LibreNMS VM  
   
```
df \-h  
sudo du \-sh /opt/librenms  
docker system df
```


Confirmed disk space usage and that LibreNMS was installed in /opt/librenms.

Step 8: Run Manual Pollers  
Used these to debug alert behavior later:  
   
```
docker exec \-it librenms    
./poller.php \-h 192.168.1.72 \-d \-m os,storage
```


\# or modern replacement:  
```
docker exec \-it librenms lnms device:poll 192.168.1.72
```


LibreNMS Was Now Fully Operational  
Web interface accessible, devices could be added using SNMP, and your SNMP service was running on the VM and test devices. All SNMP responses were validated via snmpwalk

---

*This next section is about setting up alerts, which is not too hard, and getting the alerts sent to [ntfy.sh](http://ntfy.sh) and topic Libre\_NMS so I could receive real-time alerts on my pc, phone, and on LibreNMS itself.*  

---

#### **LibreNMS \+ NTFY Alerting Integration: Detailed Summary**  

1\. LibreNMS Installation and Configuration

LibreNMS is installed and running in a Docker container on an Ubuntu VM (hosted on Proxmox).

Devices across your home lab were added using SNMP after deploying a custom setup-snmp.sh script.

The devices now poll correctly and are being monitored by LibreNMS.

2\. NTFY Setup for Push Notifications  
What You Did:  
Choose ntfy.sh as your notification backend.  
Created a topic (e.g., librenms-alerts) and verified credentials for publishing alerts via HTTP POST.  
Created an alert transport in LibreNMS to send alerts to NTFY.

Example Transport URL Format:  
   
https://\<username\>:\<password\>@ntfy.sh/\<your-topic\>

LibreNMS Transport Setup


Transport Type: Generic HTTP  
Method: POST  
Body: Left as default (or used Default PHP template)  
Name (what you see in LibreNMS):   
Api: LibreNMS (this is just a label — you can rename it)  
username  
password

Test Command (outside LibreNMS):  
To test your NTFY topic manually:  
   

```sh
curl \-u 'myuser:mypass' \-d "Test from curl" https://ntfy.sh/librenms-alerts  
```

3\. Disk Space Alert Setup and Test  
Alert Rule Details:  
Purpose: Trigger an alert when disk usage is \>= 75% and \< 85% on a specific monitored device.

Tested Device: ubuntu-test-vm at IP 192.168.1.72  
SNMP provides disk usage data from the HOST-RESOURCES-MIB.

 Test Steps:  
Created test files on ubuntu-test-vm to inflate disk usage:

fallocate \-l 3.5G \~/diskfill.test

Verified usage with:  

```sh
df \-h  
```


Output showed 84% on /, which should satisfy the alert condition.

Forced poll from the LibreNMS Docker host:  

```sh
docker exec \-it librenms lnms device:poll 192.168.1.72
```

LibreNMS Poller Output showed:

/ (FixedDisk): 79%  
Status: ALERT or NOCHG  
NTFY Notification was received on phone, desktop, and web, confirming end-to-end functionality.

Removed test files to verify alert recovery:  

```sh
rm \~/diskfill.test \~/diskfill-extra.test
```

4\. Device Uptime / Reachability Alert  
Alert Rule: Detect Down Devices \- this was a preinstalled one, just like the first one

Rule Logic:  
macros.device\_down \= Yes  
Applied To: Home Lab device group (contains all SNMP-monitored devices)  
Delay: 1 minute (device must be down for 60 seconds to trigger)  
Max Alerts: 5 (number of notifications to send)  
Interval: 5 minutes (how often to check while in alert state)  
Transport: Api: LibreNMS (which is your NTFY transport)

Test Steps:  
Powered off ubuntu-test-vm (192.168.1.72).  
Waited \~5 minutes for LibreNMS to detect unreachability via SNMP.

Alert was triggered automatically (without manual polling).  
NTFY notification was received indicating the device was down.

 Understanding Alert Polling and Status Messages  
Key Status Terms:  
Status	Meaning  
ALERT	Condition just became true — alert triggered.  
RECOVER	Condition no longer true — alert cleared.  
NOCHG	No change in alert state (still alerting or still normal).

Example (from manual poll output):  

Status: NOCHG  
→ Means the alert condition was already active and hasn't changed this cycle.

6\. Alert Testing Summary  
Tested Alert	Success?	Notes  
Disk usage on test VM	✅	Alert triggered, NTFY delivered  
Device down (uptime test)	✅	Alert triggered after poweroff, NTFY delivered  
Alert recovery (disk/uptime)	✅	Alerts cleared, recovery notification received (if configured)

Commands Reference  
\-Force Poll (from LibreNMS Docker host):  
```sh
docker exec \-it librenms lnms device:poll 192.168.1.72  
```
\-Manual Test of NTFY:  
```sh
curl \-u 'myuser:mypass' \-d "Testing NTFY" https://ntfy.sh/librenms-alerts 
```

Final Status

Functional device health and disk usage alerting  
Working real-time push notifications via NTFY  
A clean, scalable approach using SNMP, device groups, and transport mapping in LibreNMS

#### **Custom SNMP setup script**

A Custom Setup Script was run on every VM or server I wanted to add to LibreNMS for monitoring.  This is the “client” piece to the LibreNMS server piece.  This script resided on my management machine in a scripts folder, and then I simply copied it using scp to each of the VMs. I made it executable and then ran it, and it set up the SNMP client on each server or VM device. 

Here is the script to set up an SNMP client on each device to communicate TO LibreNMS

```sh
\#\!/bin/bash

\# Exit on error  
set \-e

\# Community string (change this if you're using a custom one)  
COMMUNITY="public"

\# Location and contact (optional but useful for device metadata in LibreNMS)  
SYSLOCATION="Home Lab"  
SYSCONTACT="admin@yourdomain.local"

echo "Installing SNMP daemon..."  
sudo apt update  
sudo apt install \-y snmpd

echo "Configuring SNMP..."  
sudo tee /etc/snmp/snmpd.conf \> /dev/null \<\<EOF  
rocommunity $COMMUNITY  
sysLocation $SYSLOCATION  
sysContact $SYSCONTACT  
agentAddress udp:161  
EOF

echo "Restarting SNMP service..."  
sudo systemctl restart snmpd

echo "Enabling SNMP service on boot..."  
sudo systemctl enable snmpd

\# Optional: Allow SNMP through UFW if it's enabled  
if sudo ufw status | grep \-q "Status: active"; then  
    echo "Allowing SNMP through UFW..."  
    sudo ufw allow 161/udp  
fi

echo "SNMP setup complete. Verifying..."  
snmpwalk \-v2c \-c $COMMUNITY localhost system | head \-n 5
```


