---
layout: post
draft: false
title: "Backing up docker-compose.yml files from applications running on proxmox servers to local git repo"
date: 2026-01-23
description: "This post documents the few steps I must follow each month when I do a backup of my docker-compose.yml files across my many applications running in VMs on Proxmox hypervisors. I just back them up to my local laptop and then I push them to a local Git repo that runs on its own VM"
---

## Docker Compose Backup Workflow

Complete workflow for backing up all Docker Compose YAML files to local git repository and remote backup server.

### Prerequisites
- All VMs must be running before syncing

### Steps

**Step 1:** Check all VMs are running in Proxmox GUI

**Step 2:** Run the sync script - This will check each server running on the Proxmox hypervisors for the docker-compose.yml files and copy it to the local repo on my laptop in a dated folder. *The server list is hard coded into this script, so it must be updated as needed to get all the docker-compose.yml files including any newly added ones - from new applications. 
*
```sh
~/Scripts/sync-docker-compose-repo.sh
```

**Step 3:** Go to the repo and add everything that changed
```sh
cd ~/compose-repo
git add .
```

**Step 4:** Commit with a message
```sh
git commit -m "Updated compose files [date]"
```

**Step 5:** Push to local git backup repo
```sh
git push
```

### What This Does
- Syncs all docker-compose.yml files from VMs to laptop
- Creates date-stamped snapshot in ~/compose-repo/
- Commits changes to local git repository
- Pushes backup to remote git server (192.168.1.166)

### Backup Locations
1. Each VM (working copies)
2. Laptop ~/compose-repo (git repo with history)
3. Remote server 192.168.1.166 (backup)