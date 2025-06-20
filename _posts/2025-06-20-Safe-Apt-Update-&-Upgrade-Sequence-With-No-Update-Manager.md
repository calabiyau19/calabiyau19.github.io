---
layout: post
title: "Alternative to Update Manager to avoid issues"
draft: false
date:  2025-06-20
description: Running manual updates and upgrades may be preferable to using Update Manager to avoid incorrect repos installing incompatible software.   
---





##### Background: I ran Update Manager yesterday, and at the end, it said to reboot, which was a little unusual. And I set it up to reboot, and I left for three hours before I came back to look at it again. At that point, I realized the Cinnamon desktop was broken again, which had happened before running the Update Manager.  
##### Long story short, I discovered a bad repository in Update Manager that was pulling in updates from the Jammy version of Ubuntu, not the Noble version of Ubuntu, which is what Linux Mint 22 is based on. Because it installed a specific package that was updated but was incompatible with Noble, it then proceeded to uninstall Cinnamon because Cinnamon was no longer compatible with the update it did for the wrong version of Ubuntu.  
##### So, below are the step-by-step reminders of a different way of doing the updates instead of Update Manager, so that things that should not be installed, do not get installed. There is also a corresponding Chad chat on this topic called “Cinnamon crash with update manager error 06202025”



### 🛡️ New Update Workflow (Linux Mint 22 / Ubuntu 24.04)

#### Step-by-step Commands

```sh  
  grep -r '^deb ' /etc/apt/sources.list /etc/apt/sources.list.d  
```
   - Audit your APT sources; look for any entries referencing incorrect Ubuntu versions like `jammy`, `focal`, or `bionic`.  
   - Keep only `noble` (Ubuntu 24.04) and `wilma` (Mint 22) entries.  
   - How to inspect the contents of a given .list file    
```sh  
   cat /etc/apt/sources.list.d/google-chrome.list
```
 
   - Remove others using:      
```sh  
     sudo rm /etc/apt/sources.list.d/<bad-repo>.list && sudo apt update
```
     

```sh  
sudo apt update  
```

- Refresh your local package index from your active sources.

```sh  
apt -s upgrade | grep Remv  
```  
- Simulate the upgrade.  
   - Check for any packages scheduled for removal (e.g. `cinnamon`, `mint-meta-cinnamon`).  
   ⚠️ If anything critical shows up here, **do not proceed**.

```sh  
sudo timeshift --create --comments "Pre-update snapshot" --tags D  
```

   - Create a Timeshift snapshot to allow easy rollback in case anything goes wrong.

```sh  
sudo apt upgrade  
```  
- Apply all available updates.

### Optional Enhancements

- **Show upgraded packages during apt-get operations:**

```sh
  echo 'APT::Get::Show-Upgraded "true";' | sudo tee /etc/apt/apt.conf.d/99show-upgraded  
```

