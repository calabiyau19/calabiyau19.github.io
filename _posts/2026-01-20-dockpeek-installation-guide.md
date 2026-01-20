---
layout: post
draft: false
title: "How to install DockPeek on a Ubuntu VM with many Docker containers running"
date: 2026-01-20
description: "In my home lab, most of my Docker apps run in their own VMs on Ubuntu servers, which are on Proxmox hypervisors. As a continual test, I have one VM that contains 22 Docker containers, which I believe is seven or eight individual applications. I was looking for a simple way to check this particular VM for any Docker container updates, and I ran across DockPeek, I believe, on one of the Reddit boards for self-hosting. This post details the steps I went through to install DocPeak on this particular Ubuntu VM with all these Docker containers, and TL:DR - it works great for what I needed it to do."
---


## Installing DockPeek on a Multi-Container Docker VM

### What is DockPeek?

DockPeek is a lightweight, self-hosted Docker dashboard that provides:

- **Visual overview** of all containers on a Docker host
- **Automatic update detection** - checks Docker registries for newer images
- **One-click container updates** - pull new images and recreate containers with a single button
- **Port mapping display** - see all exposed ports at a glance
- **Live container logs** - view logs in real-time through web interface
- **Container management** - start, stop, restart containers from the web UI
- **Traefik integration** - automatically detects Traefik labels for service URLs

### DockPeek vs Other Docker Management Tools

There are several popular Docker management tools available. Here's how DockPeek compares to the most widely-used options:

**Portainer** (Most Popular/Feature-Rich)
- **What it does:** Full-featured Docker/Kubernetes management platform with extensive controls
- **Strengths:** Network management, volume controls, event logs, user authentication, comprehensive container properties, Docker Swarm support, single container management
- **Weaknesses:** Heavy/complex interface, slower to load, stores compose files in proprietary structure, can be overwhelming for simple tasks
- **Best for:** Production environments, managing Docker networks, advanced users needing full control

**Dockge** (Compose-Focused Alternative)
- **What it does:** Lightweight docker-compose stack manager with clean UI
- **Strengths:** Fast loading, stores compose files on disk as standard YAML, real-time terminal output, can edit compose files directly, intuitive stack organization
- **Weaknesses:** Limited to docker-compose only, no network management, basic logging (all containers in one feed), no individual volume management
- **Best for:** Users who work primarily with docker-compose files and want simplicity

**DockPeek** (Monitoring & Updates Focused)
- **What it does:** Specialized dashboard for viewing containers and managing updates
- **Strengths:** Automatic update detection, one-click updates, clean port display, minimal setup, lightweight, excellent for multi-host monitoring
- **Weaknesses:** Limited container creation features, focused on monitoring rather than deployment, no compose file editing
- **Best for:** Users who deploy via compose files but want easy update management and port visibility

**Yacht** (Template-Based)
- **What it does:** Template-driven container deployment with app store-like interface
- **Strengths:** Pre-built templates for popular apps, user-friendly for beginners
- **Weaknesses:** Limited to template-based deployment, less flexibility for custom setups
- **Best for:** Beginners wanting quick deployment of common applications

**Key Differences Summary:**

| Tool | Primary Focus | Complexity | Update Management | Compose File Editing |
|------|---------------|------------|-------------------|---------------------|
| Portainer | Full control | High | Manual | View only (after deploy) |
| Dockge | Stack management | Medium | Manual pull | Yes (direct editing) |
| DockPeek | Monitoring & updates | Low | Automatic detection + 1-click | No |
| Yacht | Quick deployment | Low | Manual | Limited |

**When to Choose DockPeek:**

Choose DockPeek when you:
- Deploy containers using docker-compose files via SSH/command line
- Want automatic update notifications without manual checking
- Need to monitor multiple Docker hosts from one dashboard
- Prefer a lightweight, focused tool over feature-heavy platforms
- Want quick access to container ports and web interfaces
- Already have a workflow for creating containers but need easier update management

**When to Choose Portainer Instead:**

Choose Portainer when you:
- Need to create/manage Docker networks
- Require user authentication and role-based access
- Want detailed resource monitoring and event logs
- Need to manage single containers without compose files
- Require advanced volume management
- Managing production environments with complex requirements

**When to Choose Dockge Instead:**

Choose Dockge when you:
- Want to edit docker-compose files through the web interface
- Need real-time terminal output during deployments
- Prefer storing compose files in standard locations on disk
- Want a faster, more responsive interface than Portainer
- Primarily work with docker-compose stacks

**Using Multiple Tools Together:**

Many users run both DockPeek and Portainer/Dockge simultaneously:
- Use Portainer or Dockge for deploying and configuring containers
- Use DockPeek for monitoring and keeping containers updated
- Each tool serves a different purpose without conflict

### Why Install DockPeek?

For VMs running multiple Docker containers (like docker-vm with 22+ containers), DockPeek eliminates the need to:

- SSH into the VM to run `docker ps` commands
- Manually check for image updates with `docker pull`
- Run update scripts for each container individually
- Remember which ports each service uses

Instead, you get a clean web interface showing all containers, their status, available updates, and quick access to their web interfaces.

### Installation Environment

This guide covers installing DockPeek on an existing Ubuntu VM that already runs multiple Docker containers.

**Prerequisites:**
- Ubuntu VM with Docker and Docker Compose already installed
- SSH access to the VM
- Available port for DockPeek web interface (default: 3420)
- Basic understanding of docker-compose.yml files

**Example environment used in this guide:**
- VM: docker-vm at 192.168.1.xxx
- Proxmox host: NUC3 at 192.168.1.xxx
- Existing containers: 22 containers running various applications
- Access method: Direct IP (http://192.168.1.xxx:3420)

### Installation Steps

#### Step 1: Create DockPeek Directory

SSH into your Docker VM and create a dedicated directory for DockPeek configuration.

```sh
mkdir -p ~/dockpeek && cd ~/dockpeek
```

This creates a `dockpeek` directory in your home folder and changes into it.

#### Step 2: Generate a Secure SECRET_KEY

DockPeek requires a SECRET_KEY for session encryption and security. Generate a random one using OpenSSL:

```sh
openssl rand -base64 48
```

**Save this output** - you'll need it in the next step. Example output:
```
dP9mK3xR7nQ2wY8vL5jT4hB6fG1zN0cS9aM8eX2uI7oW3qA6tH5yV4rU1pJ0kD7
```

#### Step 3: Create docker-compose.yml

Create the DockPeek configuration file using nano:

```sh
nano docker-compose.yml
```

**Type the following configuration** (replace placeholders with your values):

```yaml
services:
  dockpeek:
    image: dockpeek/dockpeek:latest
    container_name: dockpeek
    environment:
      - SECRET_KEY=YOUR_GENERATED_SECRET_KEY_HERE
      - USERNAME=your_username
      - PASSWORD=your_password
      - DOCKER_HOST_NAME=your-vm-name
    ports:
      - "3420:8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```

**Configuration explanation:**

- `SECRET_KEY`: Paste the random key generated in Step 2
- `USERNAME`: Your chosen login username
- `PASSWORD`: Your chosen login password
- `DOCKER_HOST_NAME`: Display name for this Docker host (e.g., "docker-vm")
- `ports`: Exposes DockPeek on port 3420 (external) mapping to container port 8000 (internal)
- `volumes`: Gives DockPeek read-only access to Docker socket to monitor containers
- `restart`: Automatically restarts DockPeek unless manually stopped

**Save and exit nano:**
1. Press `Ctrl+O` to save
2. Press `Enter` to confirm filename
3. Press `Ctrl+X` to exit

#### Step 4: Verify Configuration

Check that the docker-compose.yml file was created correctly:

```sh
cat docker-compose.yml
```

Verify all placeholders have been replaced with your actual values.

#### Step 5: Start DockPeek

Pull the DockPeek image and start the container:

```sh
docker compose up -d
```

This command:
- Downloads the DockPeek image from Docker Hub (first run only)
- Creates the DockPeek container
- Starts it in detached mode (runs in background)

**Expected output:**
```
[+] Running 13/13
 ✔ dockpeek Pulled
 ✔ Container dockpeek Started
```

#### Step 6: Verify DockPeek is Running

Check that the DockPeek container started successfully:

```sh
docker ps | grep dockpeek
```

You should see the dockpeek container listed with status "Up" and port mapping showing 3420:8000.

#### Step 7: Access DockPeek Web Interface

Open your web browser and navigate to:

```
http://YOUR_VM_IP:3420
```

**Example:** `http://192.168.1.xxx:3420`

**Login with:**
- Username: The username you set in docker-compose.yml
- Password: The password you set in docker-compose.yml

### Using DockPeek

Once logged in, you'll see:

**Main Dashboard:**
- All containers listed with their names, status, and images
- Port numbers displayed (clickable to open services)
- Container health status indicators
- Update available badges on outdated containers

**Key Features:**

**Checking for Updates:**
1. Click "Check for Updates" button
2. DockPeek queries registries for newer images
3. Containers with available updates show an "Update" button

**Updating a Container:**
1. Click the "Update" button on any container
2. DockPeek pulls the new image
3. Stops the old container
4. Recreates container with new image
5. Starts the updated container
6. Shows real-time progress during update

**Viewing Logs:**
1. Click on any container name
2. View real-time logs in the web interface
3. No need to SSH and run `docker logs`

**Quick Access to Services:**
- Click any port number to open that service in a new tab
- Useful for services with web interfaces

### Configuration Notes

**Single VM Installation (Option A):**
This guide covers monitoring containers on a single Docker host. DockPeek accesses the local Docker socket (`/var/run/docker.sock`) to discover and manage containers.

**Multi-Host Setup (Option B - Not Covered):**
DockPeek can monitor containers across multiple Docker hosts using socket proxies. This requires:
- Installing socket proxy containers on each remote Docker host
- Adding `DOCKER_HOST_N_URL` environment variables to DockPeek configuration
- Additional security configuration

For environments where most VMs run single Docker containers, Option A (single VM monitoring) is simpler and more practical.

### Troubleshooting

**Container won't start:**
Check logs:
```sh
docker logs dockpeek
```

**Can't access web interface:**
Verify port is not already in use:
```sh
sudo netstat -tulpn | grep 3420
```

**Permission errors:**
Ensure the Docker socket is accessible:
```sh
ls -la /var/run/docker.sock
```

**Container not updating:**
Check if docker-compose.yml is in the container's directory. Some containers require being in their original directory structure to update properly.

### Maintenance

**Updating DockPeek itself:**
```sh
cd ~/dockpeek
docker compose pull
docker compose up -d
```

**Viewing DockPeek logs:**
```sh
docker logs dockpeek -f
```

**Stopping DockPeek:**
```sh
cd ~/dockpeek
docker compose down
```

**Restarting DockPeek:**
```sh
cd ~/dockpeek
docker compose restart
```

### Security Considerations

**Docker Socket Access:**
DockPeek requires read-write access to the Docker socket to manage containers. This gives it full Docker API access on the host.

**For production environments:**
- Use a socket proxy (like docker-socket-proxy) instead of direct socket access
- Limit socket proxy permissions to only what DockPeek needs
- Place DockPeek behind a reverse proxy with authentication

**For homelab/internal use:**
- Direct socket access is acceptable
- Use strong passwords
- Restrict network access to trusted devices only
- Don't expose port 3420 to the internet

### Benefits Over Manual Management

**Before DockPeek:**
- SSH into each VM
- Run `docker ps` to see containers
- Manually check for updates with `docker pull`
- Run `docker compose pull && docker compose up -d` for each service
- Keep track of which ports each service uses

**With DockPeek:**
- Open one web page
- See all containers, ports, and status at a glance
- Click "Update" button for any container
- Visual feedback during updates
- Quick access to logs without SSH

For VMs running 20+ containers, this saves significant time and reduces the chance of missing updates or forgetting which services are running.

### Conclusion

DockPeek provides a simple, efficient way to manage multiple Docker containers on a single VM through a clean web interface. The automatic update detection and one-click updates are particularly valuable for maintaining multiple services without manual scripting or SSH access.
