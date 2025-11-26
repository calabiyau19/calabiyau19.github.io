---
layout: post
title: "Docker Compose Update Process"
draft: false
date: 2025-11-21
description: "Simple to follow general rules to update docker compose installed applications like Immich, Mealie, Paperless-ngx, etc."
---


## Docker Compose Update Process

## Standard Update Procedure

### 1. Check Release Notes

- Review release notes for breaking changes
- Look for required migrations or database updates
- Check for new environment variables in `.env` file

### 2. Pull New Images

```bash
cd /path/to/docker-compose-directory
docker compose pull
```

Downloads the latest images without stopping containers.

### 3. Restart Containers

```bash
docker compose up -d
```

Stops old containers and starts new ones with updated images.

## Command Syntax Note

Use `docker compose` (space, not hyphen). The old `docker-compose` (hyphen) still works but is the legacy standalone tool.

## Version Management

### Floating Tags (Auto-update)

```yaml
image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
```

- This uses tags like `release`, `latest`, or `stable`
- Updates to the newest version when you pull
- Good for staying current automatically

### Pinned Versions (Manual control)

```yaml
image: ghcr.io/immich-app/immich-server:v2.3.1
```

- Stays on exact version until you change it
- Better for production/critical services
- Prevents surprise updates

## Best Practices

### Before Major Updates

- Ensure backups are current
- Check for special upgrade paths (e.g., v1.x → v2.x)
- Review documentation for migration steps

### After Updates

```bash
docker ps                                    # Check containers are running
docker logs <container_name>                 # Verify startup succeeded
docker logs <container_name> 2>&1 | grep -i error  # Check for errors
```

## Quick Reference

```bash
# Standard update sequence
cd ~/app/docker
docker compose pull
docker compose up -d
docker ps
docker logs <container_name>
```

## Common Applications

This process works for most containerized apps:

- Immich
- Mealie
- Paperless-ngx
- Jellyfin
- And most Docker Compose deployments
