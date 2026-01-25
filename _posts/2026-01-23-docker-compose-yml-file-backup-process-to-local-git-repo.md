---
layout: post
draft: false
title: "Backing up docker-compose.yml files from applications running on proxmox servers to local git repo"
date: 2026-01-23
description: "This post documents the few steps I must follow each month when I do a backup of my docker-compose.yml files across my many applications running in VMs on Proxmox hypervisors. I just back them up to my local laptop and then I push them to a local Git repo that runs on its own VM. This updated document also describes the automation built into the script much better."
---

## Docker Compose Backup Workflow

Automated workflow for backing up all Docker Compose YAML files to local git repository and remote backup server with visual verification.

### System Architecture

**Three-tier backup system:**
1. **Source:** Docker Compose files on individual VMs across homelab network
2. **Local:** Git repository on laptop (`~/compose-repo/`)
3. **Remote:** Backup git server at 192.168.1.166 with viewable copy

### Prerequisites

- All VMs must be running before syncing (verify in Proxmox GUI)
- Remote git server configured at 192.168.1.166
- Sync script located at `~/Scripts/sync-docker-compose-repo.sh`
- Git remote named "origin" pointing to `mark@192.168.1.166:~/git-repos/compose-repo.git`

### Complete Automated Workflow

**Step 1:** Check all VMs are running in Proxmox GUI

**Step 2:** Run the automated sync script
```sh
~/Scripts/sync-docker-compose-repo.sh
```

**That's it!** The script now automatically:
- Syncs all docker-compose.yml files from VMs to laptop
- Creates date-stamped snapshot in `~/compose-repo/YYYY-MM-DD/`
- Commits changes to local git repository
- Pushes to remote backup server (192.168.1.166)
- Updates viewable copy on backup server

### Visual Verification

**Option 1: Check git status on laptop**
```sh
cd ~/compose-repo
git status
```

Should display: "Your branch is up to date with 'origin/master'" and "nothing to commit, working tree clean"

**Option 2: Browse backup server via Nemo**
1. Open Nemo file manager
2. Connect to `sftp://192.168.1.166`
3. Navigate to `/home/mark/compose-repo-view/`
4. View all date-stamped snapshot folders

**Option 3: Command-line verification**
```sh
ssh mark@192.168.1.166 "ls -la ~/compose-repo-view/ | grep 2026"
```

### What the Script Does

The automated script performs these operations:

1. **Sync Phase:** SSH into each VM and copy docker-compose files
   - Scans standard locations: `/opt`, `/srv`, `/home`, `/docker`, `/data`
   - Copies associated `.env` files
   - Special handling for Jellyfin backup (192.168.1.64)
   - Creates organized folders: `YYYY-MM-DD/vm-name/app-name/`

2. **Git Phase:** Automatic version control
   - `git add .` - Stage all changes
   - `git commit -m "Snapshot YYYY-MM-DD: Automated sync from all VMs"`
   - `git push` - Push to remote backup server

3. **Verification Phase:** Update viewable copy
   - SSH into backup server
   - `git pull` in `~/compose-repo-view/`
   - Makes files immediately browsable

### Repository Structure
```
~/compose-repo/
├── 2025-12-18/          # December snapshot
├── 2026-01-23/          # January 23 snapshot
├── 2026-01-25/          # January 25 snapshot (current)
├── 192.168.1.x/         # Old IP-based structure (archived)
├── .git/                # Git version control
├── .gitignore           # Excludes *.log files
├── compose-sync.log     # Sync script log
└── docker-update-check.log
```

Each date-stamped folder contains:
```
2026-01-25/
├── ghostfolio-vm/
│   └── ghostfolio/
│       ├── docker-compose.yml
│       └── .env
├── immich-vm/
│   └── immich/
│       ├── docker-compose.yml
│       └── .env
├── jellyfin-vm/
│   └── etc/jellyfin/    # Full config backup
└── [other VMs...]
```

### Backup Locations

1. **Working copies:** Each VM at original locations
2. **Local git repo:** Laptop `~/compose-repo/` (with full history)
3. **Remote bare repo:** 192.168.1.166 `~/git-repos/compose-repo.git` (backup)
4. **Viewable copy:** 192.168.1.166 `~/compose-repo-view/` (for browsing)

### VM Host Mapping

The script backs up compose files from these VMs:

| IP Address | VM Name | Primary Service |
|------------|---------|-----------------|
| 192.168.1.64 | jellyfin-vm | Jellyfin media server |
| 192.168.1.65 | mealie-vm | Mealie recipe manager |
| 192.168.1.74 | proxy-vm | Reverse proxy |
| 192.168.1.98 | reactive-resume-vm | Resume builder |
| 192.168.1.108 | docker-vm | General Docker services |
| 192.168.1.110 | freshrss-vm | FreshRSS feed reader |
| 192.168.1.124 | excalidraw-vm | Excalidraw diagrams |
| 192.168.1.132 | paperless-vm | Paperless-ngx documents |
| 192.168.1.141 | immich-vm | Immich photo management |
| 192.168.1.152 | navidrome-vm | Navidrome music |
| 192.168.1.155 | ghostfolio-vm | Ghostfolio wealth tracking |
| 192.168.1.166 | git-repo-local-vm | Git backup server |
| 192.168.1.176 | librenms-vm | LibreNMS monitoring |
| 192.168.1.167 | samba-storage-vm | SMB storage |
| 192.168.1.116 | ai-hedgefund-vm | AI projects |

### Troubleshooting

**If git status shows uncommitted changes:**
```sh
cd ~/compose-repo
git add .
git commit -m "Manual commit: [describe changes]"
git push
ssh mark@192.168.1.166 "cd ~/compose-repo-view && git pull"
```

**If viewable copy is out of sync:**
```sh
ssh mark@192.168.1.166 "cd ~/compose-repo-view && git pull"
```

**If backup server connection fails:**
- Verify 192.168.1.166 is powered on in Proxmox
- Test SSH connection: `ssh mark@192.168.1.166`
- Check git remote: `cd ~/compose-repo && git remote -v`

### Script Enhancement Details

The sync script was enhanced to include these automatic operations at the end:
```sh
# Auto-commit and push to git
echo "📝 Committing changes to git..." | tee -a "$log_file"
cd "$HOME/compose-repo"
git add .
git commit -m "Snapshot $today: Automated sync from all VMs" | tee -a "$log_file"

echo "⬆️ Pushing to remote backup server..." | tee -a "$log_file"
git push | tee -a "$log_file"

echo "🔄 Updating viewable copy on backup server..." | tee -a "$log_file"
ssh mark@192.168.1.166 "cd ~/compose-repo-view && git pull" | tee -a "$log_file"

echo "✅ Full backup workflow complete!" | tee -a "$log_file"
```

### Benefits of This System

- **Automated:** Single command backs up entire infrastructure
- **Visual verification:** Browse files like a regular folder structure
- **Version history:** Full git history of all changes over time
- **Off-site backup:** Remote server protects against laptop failure
- **Date-stamped:** Easy to identify when snapshots were created
- **Comprehensive:** Includes compose files AND environment variables
- **Logged:** Complete operation log in `compose-sync.log`

### Best Practices

1. Run sync after any docker-compose.yml changes
2. Check Proxmox to ensure all VMs are running first
3. Review `compose-sync.log` for any warnings or errors
4. Keep old snapshots for historical reference
5. Clean up very old snapshots periodically to save space

### Initial Setup (Already Complete)

For reference, the initial setup included:

**On backup server (192.168.1.166):**
```sh
mkdir -p ~/git-repos
cd ~/git-repos
git init --bare compose-repo.git
git clone ~/git-repos/compose-repo.git ~/compose-repo-view
```

**On laptop:**
```sh
cd ~/compose-repo
git remote add origin mark@192.168.1.166:~/git-repos/compose-repo.git
```

This setup only needs to be done once and is already configured.