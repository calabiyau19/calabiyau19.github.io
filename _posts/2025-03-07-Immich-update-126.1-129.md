---
layout: post
title: "Update Immich 126.1 to 129.0"
date:  2025-03-07
---

Scenario:  												

Immich is running in a Proxmox server VM and has not been updated since v126.1.  I have been using Watchtower to update my other docker apps, but since Immich is moving so fast, I wanted to take the time to do it manually.  Now, I need to update it to v129. This is a docker-compose installation.    
The Proxmox server has two internal NVMe SSDs.  Immich is on NVMe1, and the Immich processed images library is on NVMe 2\. Which is a custom location, not in the defaults. All of my media was uploaded to Immich from an external HDD plugged into the Proxmox server and then mounted so that IMMICH-Go could find it inside the VM/container, which saved a tremendous amount of time uploading and processing 25K pictures and movies.  With this setup, the built-in upload function also works as designed for smaller uploads.  

Steps followed to complete the update.

1\) I read the release notes and determined it **would** break my external HDD upload location, but only one single line of code was originally added to my working docker-composec.yml file so it could be entered back into the “new” docker-compose.yml after the upgrade was complete. The original .env file could be used without any changes.    
[https://github.com/immich-app/immich/releases/tag/v1.129.0](https://github.com/immich-app/immich/releases/tag/v1.129.0)

2\) found this was not a breaking change  
[https://tinyurl.com/breaking-change](https://tinyurl.com/breaking-change)

3\) full backup using Proxmox backup tools  
4\) backup current files  
\`\`\`  
cp docker-compose.yml docker-compose.custom-backup.yml  
cp .env .env.custom-backup  
\`\`\`  
5\) reset docker-compose.yml  
\`\`\`  
git restore docker-compose.yml  
\`\`\`  
6\) verify it was reset \- I had an ownership and permission error here on my immich/docker location that I had to fix before running this command without error and continuing   
\`\`\`  
git status

\`\`\`  
7\) switch to the main branch (and found out I was 147 commits behind and NOT on the main branch after my last attempt at an update).    
\`\`\`  
git checkout main  
\`\`\`  
8\) pull the latest changes  
\`\`\`  
git pull  
\`\`\`  
9\) switch to the Latest Release (v1.129.0)  
\`\`\`  
git checkout $(git describe \--tags $(git rev-list \--tags \--max-count=1))  
\`\`\`  
10\) pull the latest docker images  
\`\`\`  
docker compose pull  
\`\`\`  
11\) restart Immich with updated version  
\`\`\`  
docker compose up \-d  
\`\`\`  
12\) verify all containers are running  
\`\`\`  
docker ps \-a  
\`\`\`  
13\) Run test uploads.  

