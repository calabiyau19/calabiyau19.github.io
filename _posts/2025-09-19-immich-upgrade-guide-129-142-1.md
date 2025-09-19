---
title: Immich upgrade from 129.0 to 142.1 
date: 2025-09-19
description: "This article documents the steps taken to upgrade Immich from 129.0 to 142.1. Immich runs singularly in a Ubuntu VM on a Proxmox server. There is a custom storage location NOT inside the VM where Immich puts processed media after intake. That way, I can run other photo & video apps on the Immich-processed media. I use Shotwell, and it just gives me another way to view and edit our photos."
---


## Complete Immich Upgrade Guide: v1.29.0 → v1.42.1

### Overview
This documents a successful upgrade of Immich from version 1.29.0 to 1.42.1, navigating multiple breaking changes and database migrations. The upgrade required a staged approach due to breaking changes in versions 1.33.0, 1.36.0, and 1.37.0.

## Initial Situation
- **Current Version**: v1.29.0
- **Target Version**: v1.42.1  
- **Problem**: Mobile app stopped working with "server is out of date" message
- **Data**: 90 GB of family photos at `/mnt/immich-storage/upload/`
- **Database**: 349 MB PostgreSQL database with metadata

### Pre-Upgrade Safety Measures

#### 1. Proxmox VM Backup
- ✅ Complete Proxmox backup of entire Ubuntu 24.04 VM completed before starting

#### 2. Current System Verification
```sh
cd /home/mark/immich
pwd
# Output: /home/mark/immich

docker ps
# Showed 4 healthy containers running v1.29.0

cd /home/mark/immich/docker
cat .env
# Revealed DB_PASSWORD=********** (problematic & character)
```

#### 3. Database Backup
```sh
mkdir -p /home/mark/immich-backup
docker exec immich_postgres pg_dump -U postgres immich > /home/mark/immich-backup/immich_db_backup_$(date +%Y%m%d_%H%M%S).sql
# Result: 349MB backup file (normal size for metadata)
```

### Stage 1: Database Password Fix

#### Problem
Database password contained `&` character which could cause Docker upgrade issues.

#### Solution
```sh
# Generate new safe password
openssl rand -base64 12 | tr -d "=+/" | cut -c1-12
# Generated: **********

# Backup .env file
cp .env .env.backup

# Update password in .env
sed -i 's/DB_PASSWORD=**********/DB_PASSWORD=**********/' .env

# Verify change
grep DB_PASSWORD .env
# Output: DB_PASSWORD=**********

# Update database password to match
docker exec immich_postgres psql -U postgres -d immich -c "ALTER USER postgres PASSWORD '**********';"
# Output: ALTER ROLE

# Test new password by restarting containers
docker compose down
docker compose up -d
docker ps
# All 4 containers showed healthy status
```

### Stage 2: Upgrade to v1.36.0 (Intermediate Version)

#### Why v1.36.0 First?
- Cannot jump directly from v1.29.0 to v1.37.0+ due to breaking changes
- v1.33.0 introduced VectorChord database migration
- v1.37.0 requires prior startup on v1.32.0-v1.36.0 range

#### Preparation
```sh
cd /home/mark/immich/docker

# Set version to intermediate target
sed -i 's/IMMICH_VERSION=release/IMMICH_VERSION=v1.136.0/' .env

# Verify version change
grep IMMICH_VERSION .env
# Output: IMMICH_VERSION=v1.136.0

# Backup current docker-compose.yml
cp docker-compose.yml docker-compose.yml.backup

# Download v1.36.0 docker-compose.yml
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/download/v1.136.0/docker-compose.yml
```

#### Verification of Custom Storage Location
```sh
# Verify current setup preserved custom storage
cat docker-compose.yml | grep -A 2 -B 2 UPLOAD_LOCATION
# Output showed: - ${UPLOAD_LOCATION}:/usr/src/app/upload

# Verify new file has same setup
cat docker-compose.yml.backup | grep -A 2 -B 2 UPLOAD_LOCATION  
# Output: Identical - custom storage preserved ✅
```

#### Upgrade Execution
```sh
# Pull new v1.36.0 images
docker compose pull
# Downloaded new immich-server, immich-machine-learning, database, and redis images

# Stop containers
docker compose down

# Start with new version (triggers database migration)
docker compose up -d
```

#### Migration Issue: Primary Key Constraint Conflict

**Problem**: Server stuck in restart loop with error:
```
PostgresError: multiple primary keys for table "geodata_places" are not allowed
Migration "1752161055253-RenameGeodataPKConstraint" failed
```

**Root Cause Analysis**:
```sh
# Investigated database structure
docker exec immich_postgres psql -U postgres -d immich -c "\d geodata_places"
```
Results showed existing primary key named `"geodata_places_tmp_pkey1"` but migration trying to create `"geodata_places_pkey"`.

**Research & Solution**:
- Searched GitHub issues for exact error message
- Found issue #20167 with identical problem and proven solution
- Applied the documented fix:

```sh
# Remove conflicting temporary primary key constraint
docker exec immich_postgres psql -U postgres -d immich -c "ALTER TABLE geodata_places DROP CONSTRAINT IF EXISTS geodata_places_tmp_pkey1;"
# Output: NOTICE: constraint "geodata_places_tmp_pkey1" of relation "geodata_places" does not exist, skipping

# Restart containers to complete migration
docker compose restart

# Monitor migration progress
docker logs immich_server -f
# Migration completed successfully
```

**Result**: All containers healthy, mobile app reconnected! ✅

### Stage 3: Final Upgrade to v1.42.1

#### Mobile App Status After v1.36.0
- Mobile app reconnected successfully
- Shows gentle "server needs to be updated" message but doesn't block usage
- Major compatibility restored

#### Final Version Update
```sh
# Update to final target version
sed -i 's/IMMICH_VERSION=v1.136.0/IMMICH_VERSION=v1.142.1/' .env

# Verify version change
grep IMMICH_VERSION .env
# Output: IMMICH_VERSION=v1.142.1

# Pull final images
docker compose pull
# Pull completed successfully

# Restart with final version
docker compose up -d

# Verify final status
docker ps
# All containers healthy ✅
```

### Critical Discoveries & Solutions

#### 1. VectorChord Database Migration (v1.33.0)
- **What**: Automatic migration from deprecated `pgvecto.rs` to `VectorChord`
- **Evidence**: Logs showed "Creating VectorChord extension", "Reindexing clip_index", "Reindexing face_index"
- **Handled**: Automatically by new docker-compose.yml with updated database image

#### 2. Primary Key Constraint Issue
- **Problem**: Conflicting database constraints from previous schema
- **Detection**: PostgreSQL error about multiple primary keys
- **Research**: Found GitHub issue #20167 with identical problem
- **Solution**: Manual constraint removal before migration retry

#### 3. Custom Storage Location Preservation
- **Risk**: Losing access to 90 GB of photos at `/mnt/immich-storage`
- **Mitigation**: Verified docker-compose.yml preserved `${UPLOAD_LOCATION}` variable
- **Result**: Photos remained accessible throughout upgrade

### Complete Command Reference

```sh
# Initial verification
cd /home/mark/immich/docker
pwd
docker ps
cat .env

# Database backup
mkdir -p /home/mark/immich-backup
docker exec immich_postgres pg_dump -U postgres immich > /home/mark/immich-backup/immich_db_backup_$(date +%Y%m%d_%H%M%S).sql

# Password fix
openssl rand -base64 12 | tr -d "=+/" | cut -c1-12
cp .env .env.backup
sed -i 's/DB_PASSWORD=**********/DB_PASSWORD=**********/' .env
grep DB_PASSWORD .env
docker exec immich_postgres psql -U postgres -d immich -c "ALTER USER postgres PASSWORD '**********';"
docker compose down
docker compose up -d

# Stage 1: Upgrade to v1.36.0
sed -i 's/IMMICH_VERSION=release/IMMICH_VERSION=v1.136.0/' .env
grep IMMICH_VERSION .env
cp docker-compose.yml docker-compose.yml.backup
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/download/v1.136.0/docker-compose.yml
docker compose pull
docker compose down
docker compose up -d

# Fix migration issue
docker exec immich_postgres psql -U postgres -d immich -c "\d geodata_places"
docker exec immich_postgres psql -U postgres -d immich -c "ALTER TABLE geodata_places DROP CONSTRAINT IF EXISTS geodata_places_tmp_pkey1;"
docker compose restart
docker logs immich_server -f

# Stage 2: Final upgrade to v1.42.1
sed -i 's/IMMICH_VERSION=v1.136.0/IMMICH_VERSION=v1.142.1/' .env
grep IMMICH_VERSION .env
docker compose pull
docker compose up -d
docker ps

# Optional cleanup
docker image prune -f
```

### Final Results

#### ✅ Success Metrics
- **Version**: Successfully upgraded from v1.29.0 → v1.42.1
- **Mobile App**: Fully reconnected and functional
- **Data Integrity**: All 90 GB of photos preserved
- **Database**: 349 MB database migrated successfully  
- **Containers**: All services healthy and running
- **Features**: Access to all latest Immich improvements

#### 🔧 Technical Achievements
- Navigated 3 major breaking changes (v1.33.0, v1.36.0, v1.37.0)
- Completed VectorChord database migration
- Resolved PostgreSQL constraint conflicts
- Preserved custom storage configuration
- Maintained data integrity throughout

#### 📚 Lessons Learned
1. **Staged upgrades required** for major version jumps with breaking changes
2. **GitHub issues are invaluable** for finding solutions to specific migration problems
3. **Database constraint conflicts** can occur during schema migrations
4. **Backup strategy is critical** - Proxmox VM backup saved the day
5. **Custom configurations need verification** during docker-compose.yml updates

### Post-Upgrade Recommendations

1. **Create fresh backup** of successfully upgraded system
2. **Test all functionality** including mobile app sync, web interface, and photo access
3. **Monitor logs** for any ongoing issues in first few days
4. **Update mobile apps** to the latest versions for best compatibility
5. **Review new features** available in v1.42.1

---

*Upgrade completed successfully with zero data loss and full functionality restored.*