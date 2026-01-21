---
layout: post
draft: false
title: "Generic Procedures to update Docker Compose installed applications on Ubuntu server VMs"
date: 2026-01-21
description: "There is an older, shorter version of this process on Pontifex.site but the process keeps getting refined as I acquire more information about best practices,and improved documentation methods, and this post documents the most up to date version of that process."
---


## Generic Docker Compose Service Update Procedure

#### NOTE: to see if new docker containers exist for your project, so you can update them, run these commands inside the directory containing the docker-compose.yml file:

```sh
docker compose pull
```
Or to be extra careful, specify the specific docker container name - which you can get by running docker ps. 
```sh
docker compose -p speedtest-tracker pull
```

### TL;DR
```sh
docker compose pull
```
```sh
docker compose up -d
```
```sh
docker ps	  
```
```sh
docker system prune
```


## Overview
This document provides a standardized procedure for updating Docker services running with docker-compose.yml files on Ubuntu VMs in Proxmox.

## Applicable Services
This procedure applies to any Docker Compose-based service including:
- Homepage
- Immich
- LibreNMS
- Jellyfin
- Ghostfolio
- Mealie
- Paperless-ngx
- Any other containerized service

## Pre-Update Checklist

- [ ] Identify current version of the service
- [ ] Check release notes for breaking changes
- [ ] Verify backup status
- [ ] Schedule maintenance window (if applicable)
- [ ] Have SSH access to the VM
- [ ] Know the location of docker-compose.yml file

## Update Procedure

### Step 1: Review Release Notes

Before updating, **always** review the change log:

```bash
# Most projects have releases on GitHub
# Example format:
https://github.com/[organization]/[project]/releases
```

**Look for:**
- Breaking changes
- Required migration steps
- New environment variables
- Configuration changes
- Known issues

**Common sources:**
- GitHub releases page
- Project documentation
- Docker Hub change log
- Project website/blog

### Step 2: Create Proxmox VM Snapshot

**This is mandatory** - snapshots provide instant rollback capability.

**Via Proxmox Web Interface:**
1. Select the VM in Proxmox
2. Navigate to **Snapshots** tab
3. Click **Take Snapshot**
4. Name format: `before-[service]-[version]-[date]`
   - Example: `before-immich-2.0.1-20241214`
5. Add description noting current version
6. Wait for completion (usually 10-30 seconds)

**Via Proxmox CLI:**
```bash
# On Proxmox host
qm snapshot [VMID] before-[service]-update --description "Before updating [service] to [version]"
```

### Step 3: Connect to the VM

```bash
ssh [VM-IP-ADDRESS]
```

Example:
```bash
ssh 192.168.1.108
```

### Step 4: Document Current Configuration

**Locate docker-compose.yml:**

```bash
# Common locations:
ls -la /home/*/docker/*/docker-compose.yml
ls -la /opt/docker/*/docker-compose.yml
ls -la ~/docker-compose.yml
```

**Inspect running container:**

```bash
# List running containers
docker ps

# Get detailed configuration
docker inspect [container-name]

# View mounts
docker inspect [container-name] --format='{{json .Mounts}}' | jq

# View environment variables
docker inspect [container-name] --format='{{json .Config.Env}}' | jq

# View restart policy
docker inspect [container-name] --format='{{json .HostConfig.RestartPolicy}}'
```

**Save current configuration:**

```bash
# Optional: Backup docker-compose.yml
cp docker-compose.yml docker-compose.yml.backup-$(date +%Y%m%d)
```

### Step 5: Navigate to Service Directory

```bash
cd /path/to/service/docker-compose-directory/
```

Examples:
```bash
cd /home/mark/docker/homepage/
cd /opt/docker/immich/
cd ~/jellyfin/
```

### Step 6: Review docker-compose.yml

```bash
cat docker-compose.yml
```

**Check image tag:**
- `latest` - Always pulls the newest version
- `v1.2.3` - Pinned to specific version
- `stable` - Follows stable channel
- `2` - Follows major version

**Note the current tag** before updating.

### Step 7: Pull New Image(s)

**For single service:**

```bash
docker compose pull
```

**For specific service in multi-container setup:**

```bash
docker compose pull [service-name]
```

**Manually pull specific version (if needed):**

```bash
docker pull [image]:[tag]
```

**Expected output:**
```
[+] Running 1/1
 ✔ [service] Pulled
```

### Step 8: Check for Migration Requirements

Some services require database migrations or configuration updates:

```bash
# Check service documentation for migration steps
# Examples:

# Immich often requires migrations between major versions
# Paperless-ngx may need to run migrations
# LibreNMS requires database updates
```

**Common migration patterns:**

```bash
# Run one-time migration command
docker compose run --rm [service] migrate

# Or via docker exec
docker exec [container] /app/migrate.sh
```

### Step 9: Update the Service

**Standard update (recreates containers):**

```bash
docker compose up -d
```

**What this does:**
- Stops existing container(s)
- Removes old container(s)
- Creates new container(s) with updated image
- Preserves volumes and networks
- Starts container(s) in detached mode

**Force recreate (if compose doesn't detect changes):**

```bash
docker compose up -d --force-recreate
```

**For specific service only:**

```bash
docker compose up -d [service-name]
```

### Step 10: Verify Service Health

**Check container status:**

```bash
docker ps | grep [service-name]
```

Look for:
- Status: "Up" (not "Restarting")
- Health: "healthy" (if health checks configured)

**View real-time logs:**

```bash
docker compose logs -f
```

Or specific service:

```bash
docker compose logs -f [service-name]
```

Press `Ctrl+C` to exit log view.

**Check last 50 log lines:**

```bash
docker compose logs --tail 50
```

**Look for errors:**

```bash
docker compose logs | grep -i error
docker compose logs | grep -i fail
docker compose logs | grep -i fatal
```

### Step 11: Test Service Functionality

**Web-based services:**
```bash
# Access via browser
http://[VM-IP]:[PORT]
```

**Verify:**
- [ ] Service loads without errors
- [ ] Login works (if applicable)
- [ ] Main features function correctly
- [ ] Data is intact and accessible
- [ ] API endpoints respond (if applicable)
- [ ] Background jobs running (if applicable)

**CLI-based verification:**

```bash
# Example: Test API endpoint
curl http://localhost:[PORT]/api/health

# Example: Check database connection
docker compose exec [service] /check-db.sh
```

### Step 12: Clean Up Old Images (Optional)

After successful update, remove old images to free space:

```bash
# List all images
docker images

# Remove specific old image
docker rmi [image]:[old-tag]

# Or clean up all unused images
docker image prune -a
```

**Warning:** Only do this after confirming the update works!

## Rollback Procedures

### Option 1: Proxmox Snapshot Rollback (Fastest)

**Use when:**
- Service won't start
- Data corruption detected
- Critical functionality broken

**Steps:**
1. Stop the VM from Proxmox interface
2. Navigate to VM → **Snapshots** tab
3. Select the pre-update snapshot
4. Click **Rollback**
5. Confirm the rollback
6. Start the VM

**Result:** Complete system state restored to pre-update.

### Option 2: Docker Image Downgrade (Targeted)

**Use when:**
- Only the service needs rollback
- VM configuration changes should be kept
- Faster than full VM rollback

**Steps:**

```bash
# Navigate to service directory
cd /path/to/service/

# Edit docker-compose.yml
nano docker-compose.yml

# Change image tag back to previous version
# From: image: service:latest
# To:   image: service:v1.2.3

# Stop and remove containers
docker compose down

# Pull the old version
docker compose pull

# Start with old version
docker compose up -d

# Verify
docker compose logs -f
```

### Option 3: Emergency Recovery

**If service is completely broken:**

```bash
# Stop everything
docker compose down

# Remove volumes (WARNING: DATA LOSS)
docker compose down -v

# Restore from backup (if available)
# Then recreate fresh
docker compose up -d
```

## Best Practices

### Timing
- Update during low-usage periods
- Avoid updating multiple services simultaneously
- Test updates in development/staging first (if available)

### Documentation
- Keep a change log of updates in `/home/[user]/update-log.txt`
- Note version numbers before and after
- Document any issues encountered

### Backup Verification
- Test backup restoration periodically
- Verify automated backups are running
- Keep multiple snapshot/backup generations

### Version Pinning
Consider pinning to specific versions instead of `latest`:

```yaml
# Instead of:
image: immich:latest

# Use:
image: immich:v2.0.1
```

**Advantages:**
- Predictable updates
- Easier rollback
- Better change control

**Disadvantages:**
- Manual version tracking required
- May miss security patches

## Service-Specific Notes

### Immich
- Check for database migrations between major versions
- Review [migration guide](https://immich.app/docs/install/upgrade)
- May require PostgreSQL updates

### LibreNMS
- Often requires database schema updates
- Run: `docker exec librenms /opt/librenms/daily.sh`
- Verify poller is running

### Jellyfin
- Usually straightforward updates
- May need to rebuild library metadata
- Check plugin compatibility

### Paperless-ngx
- May require document re-indexing
- Check for OCR engine updates
- Verify consumption folder permissions

### Ghostfolio
- Database migrations may be required
- Check portfolio calculation accuracy post-update

## Troubleshooting

### Container Won't Start

```bash
# Check detailed logs
docker compose logs [service]

# Verify image was pulled
docker images | grep [service]

# Check resource usage
docker stats

# Verify port availability
sudo netstat -tulpn | grep [PORT]
```

### Permission Errors

```bash
# Check volume permissions
ls -la /path/to/volume/

# Fix permissions (adjust user:group as needed)
sudo chown -R 1000:1000 /path/to/volume/

# Check SELinux (if applicable)
getenforce
```

### Database Connection Issues

```bash
# Check database container
docker compose ps

# Verify database is ready
docker compose logs [db-service] | grep ready

# Test database connection
docker compose exec [db-service] pg_isready  # PostgreSQL
docker compose exec [db-service] mysqladmin ping  # MySQL
```

### Network Issues

```bash
# List networks
docker network ls

# Inspect network
docker network inspect [network-name]

# Recreate network (be cautious)
docker network rm [network-name]
docker network create [network-name]
```

### Out of Disk Space

```bash
# Check disk usage
df -h

# Clean Docker system
docker system df
docker system prune -a

# Remove old volumes (careful!)
docker volume ls
docker volume prune
```

## Automation Considerations

### Update Notification Tools
- 
- [Diun](https://crazymax.dev/diun/) - Docker Image Update Notifier
- [Watchtower](https://containrrr.dev/watchtower/) - Automated updates (use with caution)
- [Portainer](https://www.portainer.io/) - GUI for Docker management

### Monitoring
- Set up health checks in docker-compose.yml
- Use Uptime Kuma or similar for service monitoring
- Configure alerting for service downtime

## Emergency Contacts

- Service documentation: Check GitHub/official site
- Docker logs location: `/var/lib/docker/containers/[container-id]/[container-id]-json.log`
- Proxmox backups: `/var/lib/vz/dump/` (on Proxmox host)

## Standard Update Command Summary

```bash
# Quick reference for standard update
cd /path/to/service/
docker compose pull
docker compose up -d
docker compose logs -f
```

## Checklist Template

```markdown
## Update Checklist - [Service Name] - [Date]

- [ ] Current version documented: ______
- [ ] Release notes reviewed
- [ ] Breaking changes identified: Yes / No
- [ ] Proxmox snapshot created: [Snapshot Name]
- [ ] docker-compose.yml location: ______
- [ ] Configuration backed up
- [ ] Image pulled successfully
- [ ] Migration steps completed (if any)
- [ ] Service updated
- [ ] Logs checked for errors
- [ ] Functionality verified
- [ ] Old images cleaned up
- [ ] Update logged in documentation
- [ ] Snapshot deleted (after 7 days)

## Issues Encountered
[None / List any issues]

## New version: ______
```

Save this checklist for each update to maintain an audit trail.
