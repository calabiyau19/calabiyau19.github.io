---
layout: post
title: "BookLore Installation on Ubuntu VM"
date: 2025-12-24
categories: [docker, ebooks, self-hosted]
tags: [booklore, calibre-web]
---

# BookLore Installation Guide

**Date:** December 24, 2024  
**System:** Ubuntu VM (192.168.1.108) on Proxmox  
**Purpose:** Replace aging Calibre-web installation with modern BookLore ebook library manager

---

## Overview

BookLore is a modern, self-hosted ebook library management system with features including:
- Smart organization with custom shelves and magic shelves (rule-based collections)
- Multi-user support with granular permissions
- Auto metadata fetching from Goodreads, Amazon, Google Books, and Hardcover
- Kobo and KOReader device sync
- BookDrop auto-import folder
- OPDS support for reading apps
- Built-in reader for EPUB, PDF, and comics
- Flexible authentication (local or OIDC providers)

## System Information

**VM Details:**
- **Host:** 192.168.1.108
- **OS:** Ubuntu (Debian-based)
- **Platform:** Proxmox hypervisor
- **Docker:** Installed and configured
- **Timezone:** America/New_York

**Installation Directory Structure:**
```
/home/mark/docker/booklore/
├── docker-compose.yml    # Main Docker Compose configuration
├── .env                  # Environment variables and credentials
├── data/                 # BookLore application data
├── bookdrop/             # Auto-import folder for new books
└── config/
    └── mariadb/          # MariaDB database files
```

**Existing Books Location:**
- Books already exist at `/data/compose/2/books/` (from previous Calibre-web installation)
- This location is on an external SSD attached to Proxmox host
- Future plans: Migrate to standardized mount points

---

## Installation Steps

### Step 1: Create Directory Structure

SSH into the VM and create the BookLore directory structure:

```bash
ssh 192.168.1.108
mkdir -p /home/mark/docker/booklore/{data,config/mariadb,bookdrop}
cd /home/mark/docker/booklore
```

This creates:
- `/home/mark/docker/booklore/data` - BookLore application data
- `/home/mark/docker/booklore/config/mariadb` - MariaDB database files  
- `/home/mark/docker/booklore/bookdrop` - Auto-import folder

### Step 2: Create Environment File

Create `.env` file with database credentials and configuration:

```bash
nano .env
```

Add the following content:

```env
# BookLore Application Settings
APP_USER_ID=1000
APP_GROUP_ID=1000
TZ=America/New_York
BOOKLORE_PORT=6060

# Database Connection (BookLore)
DATABASE_URL=jdbc:mariadb://mariadb:3306/booklore
DB_USER=booklore
DB_PASSWORD=BookLore2025SecurePass!

# MariaDB Container Settings
DB_USER_ID=1000
DB_GROUP_ID=1000
MYSQL_ROOT_PASSWORD=MariaDBRoot2025SecurePass!
MYSQL_DATABASE=booklore
```

**Important Notes:**
- Change the passwords to secure values
- `APP_USER_ID` and `APP_GROUP_ID` set to 1000 to match existing user permissions
- Timezone set to `America/New_York` to match existing services

Save and exit: `Ctrl+X`, `Y`, `Enter`

### Step 3: Create Docker Compose File

Create `docker-compose.yml` configuration:

```bash
nano docker-compose.yml
```

Add the following content:

```yaml
services:
  booklore:
    image: booklore/booklore:latest
    container_name: booklore
    environment:
      - USER_ID=${APP_USER_ID}
      - GROUP_ID=${APP_GROUP_ID}
      - TZ=${TZ}
      - DATABASE_URL=${DATABASE_URL}
      - DATABASE_USERNAME=${DB_USER}
      - DATABASE_PASSWORD=${DB_PASSWORD}
      - BOOKLORE_PORT=${BOOKLORE_PORT}
    depends_on:
      mariadb:
        condition: service_healthy
    ports:
      - "${BOOKLORE_PORT}:${BOOKLORE_PORT}"
    volumes:
      - ./data:/app/data
      - /data/compose/2/books:/books
      - ./bookdrop:/bookdrop
    restart: unless-stopped

  mariadb:
    image: lscr.io/linuxserver/mariadb:11.4.5
    container_name: booklore-mariadb
    environment:
      - PUID=${DB_USER_ID}
      - PGID=${DB_GROUP_ID}
      - TZ=${TZ}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./config/mariadb:/config
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mariadb-admin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 10
```

**Key Configuration Details:**
- BookLore uses port 6060 (no conflict with Calibre-web on 8083)
- Books volume maps to existing Calibre-web location: `/data/compose/2/books`
- MariaDB container renamed to `booklore-mariadb` to avoid conflicts
- Health check ensures MariaDB is ready before BookLore starts
- Restart policy: `unless-stopped` (survives reboots)

Save and exit: `Ctrl+X`, `Y`, `Enter`

### Step 4: Start the Containers

Pull images and start both containers:

```bash
docker compose up -d
```

**What happens:**
- Downloads BookLore and MariaDB images (first time only)
- Creates Docker network `booklore_default`
- Starts MariaDB container and waits for it to be healthy
- Starts BookLore container
- Initial startup takes 1-2 minutes for database initialization

### Step 5: Verify Containers Are Running

Check container status:

```bash
docker compose ps
```

Expected output:
```
NAME                IMAGE                                STATUS
booklore            booklore/booklore:latest             Up 2 minutes
booklore-mariadb    lscr.io/linuxserver/mariadb:11.4.5   Up 2 minutes (healthy)
```

Wait until MariaDB shows "(healthy)" before proceeding.

---

## Initial Setup and Configuration

### Step 6: Access BookLore Web Interface

Open browser and navigate to:
```
http://192.168.1.108:6060
```

### Step 7: Create Admin Account

On the setup wizard page, fill in:
- **Username:** Your admin username
- **Full Name:** Your name
- **Email:** Your email address
- **Password:** Strong, unique password

Click **"Create Admin Account"**

### Step 8: Create First Library

After login, you'll see the dashboard with "Welcome to BookLore! Let's create your first library"

1. Click the green **"+ Create Your Library"** button

2. Fill in library configuration:
   - **Name:** "My Books" (or your preferred name)
   - **Library Icon:** Select any icon
   - **Monitor Folders:** Enable/Check this option

3. Click **"Continue to Directories"** or **"Next"**

4. Add directory path:
   ```
   /books
   ```

5. Click **"Add Directory"** or **"Create Library"**

### Step 9: Initial Book Scan

BookLore will immediately begin scanning and processing your existing books:

**What BookLore does during scan:**
- Scans all book files in `/books` directory
- Extracts embedded metadata from files
- Fetches additional metadata from online sources (if configured)
- Generates cover thumbnails
- Indexes all books in the database

**Progress:**
- Processing happens in background
- You can navigate around BookLore while it works
- Check progress in sidebar (book count will increase)
- Initial scan time varies based on collection size

---

## Container Management

### View Logs

```bash
# All containers
docker compose logs -f

# BookLore only
docker compose logs -f booklore

# MariaDB only
docker compose logs -f mariadb
```

### Stop Containers

```bash
docker compose down
```

### Restart Containers

```bash
docker compose restart
```

### Update BookLore

```bash
# Pull latest image
docker compose pull

# Recreate containers with new image
docker compose up -d
```

---

## Configuration Notes

### Database

**MariaDB Configuration:**
- Version: 11.4.5 (LTS)
- Database name: `booklore`
- Data stored in: `/home/mark/docker/booklore/config/mariadb`
- Automatic health checks ensure availability

**Backup Database:**
```bash
# Backup MariaDB config directory
cp -r /home/mark/docker/booklore/config/mariadb /path/to/backup/
```

### BookDrop Auto-Import

The BookDrop folder at `/home/mark/docker/booklore/bookdrop/` provides automatic import:

1. Copy book files to the BookDrop folder:
   ```bash
   cp /path/to/new/book.epub /home/mark/docker/booklore/bookdrop/
   ```

2. BookLore automatically:
   - Detects new files
   - Extracts metadata
   - Fetches additional information (if enabled)
   - Makes books available for review/import in the UI

3. Review and finalize imports through BookLore web interface

### Port Configuration

BookLore runs on port 6060:
- Access URL: `http://192.168.1.108:6060`
- No port conflict with existing Calibre-web (port 8083)

To change port, edit `.env` file:
```env
BOOKLORE_PORT=8080  # Change to desired port
```

Then restart containers:
```bash
docker compose down
docker compose up -d
```

---

## Migrating from Calibre-web

### Current State

**Calibre-web Status:**
- Running on same VM (192.168.1.108)
- Port: 8083
- Books location: `/data/compose/2/books/`
- Container: `calibre-web`
- Network: Connected to `calibre-web_default` and `homepage_net`

**Migration Notes:**
- Both systems can run simultaneously (different ports)
- BookLore is already reading from same books directory
- Original Calibre metadata.db remains untouched
- BookLore creates its own database in MariaDB

### Stopping Calibre-web (When Ready)

Once BookLore is fully configured and tested:

```bash
# Stop and remove Calibre-web container
docker stop calibre-web
docker rm calibre-web

# Optional: Remove Calibre-web network if not used by other containers
docker network rm calibre-web_default
```

**Calibre-web Data Locations:**
- Books: `/data/compose/2/books/` (keep for BookLore)
- Config: `/data/compose/2/config/` (can archive)
- Missing docker-compose.yml (was deleted previously)

---

## Future Improvements

### Planned Changes

1. **Standardize Mount Points:**
   - Current: Books at `/data/compose/2/books/` (external SSD via Proxmox)
   - Future: Align with standard mount point configuration used by other services
   - Will require updating docker-compose.yml volume mappings

2. **Metadata Configuration:**
   - Enable auto-metadata fetching
   - Configure Google Books API key for better metadata
   - Set up preferred metadata sources

3. **User Management:**
   - Add additional users with appropriate permissions
   - Configure reading preferences per user

4. **OPDS Integration:**
   - Configure OPDS for e-reader access
   - Test with Kobo device sync

5. **Backup Strategy:**
   - Automate database backups
   - Document restore procedures

---

## Troubleshooting

### Container Won't Start

Check logs:
```bash
docker compose logs booklore
docker compose logs mariadb
```

Common issues:
- Port 6060 already in use: Change `BOOKLORE_PORT` in `.env`
- MariaDB not healthy: Wait longer or check MariaDB logs
- Permission issues: Verify USER_ID and GROUP_ID match file ownership

### Can't Access Web Interface

1. Verify containers are running:
   ```bash
   docker compose ps
   ```

2. Check port binding:
   ```bash
   docker port booklore
   ```

3. Test connectivity:
   ```bash
   curl http://localhost:6060
   ```

### Books Not Appearing

1. Verify volume mount:
   ```bash
   docker exec booklore ls -la /books
   ```

2. Check file permissions:
   ```bash
   ls -la /data/compose/2/books/
   ```

3. Trigger manual library scan in BookLore UI:
   - Navigate to library settings
   - Click "Scan Library"

### Database Connection Issues

1. Check MariaDB is healthy:
   ```bash
   docker compose ps
   ```

2. Verify database credentials in `.env` match docker-compose.yml

3. Test database connection:
   ```bash
   docker exec booklore-mariadb mariadb -u booklore -p
   ```

---

## Additional Resources

**Official Documentation:**
- Getting Started: https://booklore-app.github.io/booklore-docs/docs/getting-started/
- Installation Guide: https://booklore-app.github.io/booklore-docs/docs/installation/
- GitHub Repository: https://github.com/booklore-app/booklore

**Docker Images:**
- Docker Hub: https://hub.docker.com/r/booklore/booklore
- GitHub Container Registry: https://ghcr.io/booklore-app/booklore

**Community:**
- Discord: https://discord.gg/Ee5hd458Uz
- GitHub Issues: https://github.com/booklore-app/booklore/issues

**Live Demo:**
- URL: https://demo.booklore.dev
- Username: `booklore`
- Password: `9HC20PGGfitvWaZ1`

---

## Version Information

**Installation Date:** December 24, 2024  
**BookLore Version:** v1.15.0 (as of installation)  
**MariaDB Version:** 11.4.5  
**Docker Compose Version:** 2.x  

---

## Notes

- Installation completed successfully on first attempt
- All existing Calibre-web books (EPUB, PDF) discovered and indexed
- Performance: Initial scan of library completed efficiently
- Future consideration: Migrate books to standardized storage mount points
- BookLore runs alongside existing Calibre-web without conflicts
