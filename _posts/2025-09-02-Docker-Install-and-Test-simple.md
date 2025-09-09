---
layout: post
title: "Docker Install and setup on Debian/Ubuntu"
draft: false
date:  2025-09-04
description: This appears in other articles on this site but since it is used in most of my new VMs that I spin up on Proxmox, it made sense to break out the instructions in a separate article.
---


## Complete Docker Setup on Ubuntu/Debian

**1. Update the system:**
```sh
sudo apt update && sudo apt upgrade -y
```

**2. Install Docker:**
```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

**3. Add user to docker group:**
```sh
sudo usermod -aG docker $USER
```

**4. Install Docker Compose:**
```sh
sudo apt install docker-compose-plugin -y
```

**5. Enable Docker service:**
```sh
sudo systemctl enable docker
sudo systemctl start docker
```

**6. Log out and back in** (or restart) to apply group changes.

**7. Verify installation:**
```sh
docker --version
docker run hello-world
```

## Notes

- The group membership change requires a logout/login to take effect
- After step 7, you should be able to run Docker commands without `sudo`
- If `hello-world` runs successfully, Docker is properly installed and configured