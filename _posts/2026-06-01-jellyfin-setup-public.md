---
layout: post
draft: false
title: Jellyfin Setup 2nd Install
date: 2026-06-01
description: "This post documents the steps to install Jellyfin a second time since the first installation somehow had a corrupted database and no matter what was tried it could not be recovered and...honestly I wasn't using it for many months. So possibly something changed and I did not realize it at the time.."

---



## Jellyfin Fresh Install & Setup — jellyfin-vm ([JELLYFIN-VM-IP])

**VM:** jellyfin-vm, [JELLYFIN-VM-IP], hosted on sv-proxmox (NUC5)
**OS:** Ubuntu 24.04 LTS (Noble)

---

## Background

Jellyfin was previously installed on this VM using an old install script. It crashed with a stack overflow (`BaseItem.get_OfficialRatingForComparison()`) and core dump (`signal=ABRT`) whenever a media library was added. Multiple recovery attempts failed. The decision was made to fully remove Jellyfin and do a clean install using the current official method.

---

## Step 1 — Verify Clean Removal of Previous Jellyfin Install

Before starting the fresh install, confirm all Jellyfin packages and data are gone.

```sh
dpkg -l | grep -i jellyfin
```

Any packages showing `rc` (removed, config retained) or `ii` (fully installed) need to be cleaned up.

```sh
sudo apt purge jellyfin jellyfin-server jellyfin-web jellyfin-ffmpeg7 -y
```

Verify nothing remains:

```sh
dpkg -l | grep -i jellyfin
```

Expected: no output.

Also confirm config directories are gone:

```sh
ls /var/lib/jellyfin /etc/jellyfin 2>&1
```

Expected: `No such file or directory` for both.

---

## Step 2 — Install Jellyfin Using Official Script

Source: `https://jellyfin.org/docs/general/installation/linux`

Download the install script and verify its checksum before running anything:

```sh
curl -s https://repo.jellyfin.org/install-debuntu.sh -O && \
curl -s https://repo.jellyfin.org/install-debuntu.sh.sha256sum -O && \
sha256sum -c install-debuntu.sh.sha256sum
```

Expected output: `install-debuntu.sh: OK`

Optionally inspect the script before running:

```sh
less install-debuntu.sh
```

Press `q` to exit. Then run the install:

```sh
sudo bash install-debuntu.sh
```

The script detects Ubuntu Noble (24.04), sets up the Jellyfin apt repository, installs all packages, starts the service, and waits 15 seconds to confirm it is running. At completion it prints the service status and the URL to access the web UI.

---

## Step 3 — Verify Installation

Confirm all four packages installed cleanly (`ii` status):

```sh
dpkg -l | grep -i jellyfin
```

Expected packages:
- `jellyfin` — meta package
- `jellyfin-server` — server binary
- `jellyfin-web` — web UI
- `jellyfin-ffmpeg7` — transcoding

Confirm the service is running:

```sh
sudo systemctl status jellyfin
```

Look for `Active: active (running)` and `Core startup complete` in the log output.

---

## Step 4 — Initial Web UI Setup

Open a browser and navigate to:

```
http://[JELLYFIN-VM-IP]:8096
```

If prompted with a "Server Mismatch" warning (because the browser previously connected to an older Jellyfin instance at this address), click **Connect Anyway**. This is expected after a fresh install.

Walk through the setup wizard:
- Set admin username and password
- On the "Set up your media libraries" step — click **Next** without adding anything yet

---

## Step 5 — Create Permanent Media Directory

All local movies are stored at `/srv/jellyfin/movies/`. This directory is owned by the `jellyfin` user with group write permissions so both the service and the `mark` user can access it.

Create the directory:

```sh
sudo mkdir -p /srv/jellyfin/movies
```

Set ownership:

```sh
sudo chown -R jellyfin:jellyfin /srv/jellyfin/movies
```

Set permissions (owner and group can read/write/execute):

```sh
sudo chmod 775 /srv/jellyfin/movies
```

Add `mark` to the `jellyfin` group so file copies via `scp` work:

```sh
sudo usermod -aG jellyfin mark
```

Log out and back in to jellyfin-vm for the group change to take effect.

---

## Step 6 — Copy Movies from Management Laptop

From lpt-HP ([MANAGEMENT-LAPTOP-IP]), copy movie files to jellyfin-vm using `scp`:

```sh
scp "/home/mark/Downloads/Movie Name.mkv" mark@[JELLYFIN-VM-IP]:/srv/jellyfin/movies/
```

Note: quotes are required if the filename contains spaces.

---

## Step 7 — Mount Samba Share (YouTube Downloads)

The samba-storage-vm ([SAMBA-STORAGE-VM-IP]) hosts YouTube downloads at `/storage/YouTube-downloads`.

Create the mount point:

```sh
sudo mkdir -p /mnt/samba-storage
```

Mount the share:

```sh
sudo mount -t cifs //[SAMBA-STORAGE-VM-IP]/storage -o username=mark,password='[SAMBA-PASSWORD]' /mnt/samba-storage
```

Note: the `&` in the password requires single quotes around the password value.

Verify the mount:

```sh
ls /mnt/samba-storage
```

Expected: `lost+found/  YouTube-downloads/`

Verify the `jellyfin` user can read the share:

```sh
sudo -u jellyfin ls /mnt/samba-storage/YouTube-downloads/
```

---

## Step 8 — Add Media Libraries in Jellyfin

### Movies Library (local files)

1. Dashboard → **Libraries** → **Add Media Library**
2. Content type: **Movies**
3. Folder path: `/srv/jellyfin/movies`
4. Click **OK** → **Save**

Jellyfin will scan the directory and attempt metadata matching against online databases.

### YouTube Downloads Library (Samba share)

1. Dashboard → **Libraries** → **Add Media Library**
2. Content type: **Home Videos**
   - Use **Home Videos** (not Movies) — this skips online metadata matching and displays files as-is, which is appropriate for YouTube downloads that have no IMDb entries
3. Folder path: `/mnt/samba-storage/YouTube-downloads`
4. Click **OK** → **Save**

---

## Step 9 — Make Samba Mount Persistent (fstab)

The manual mount command only lasts until the next reboot. To make it permanent, add an entry to `/etc/fstab`.

Add the following line:

```sh
echo '//[SAMBA-STORAGE-VM-IP]/storage  /mnt/samba-storage  cifs  username=mark,password=[SAMBA-PASSWORD],uid=jellyfin,gid=jellyfin,file_mode=0664,dir_mode=0775,nofail  0  0' | sudo tee -a /etc/fstab
```

Options explained:
- `uid=jellyfin,gid=jellyfin` — files are owned by the jellyfin service user when mounted
- `file_mode=0664,dir_mode=0775` — jellyfin and group members can read/write
- `nofail` — VM still boots normally if the Samba server is unreachable

After editing fstab, reload systemd and mount:

```sh
sudo systemctl daemon-reload
```

```sh
sudo mount /mnt/samba-storage
```

Note: if `mount -a` or `sudo mount /mnt/samba-storage` returns "device or resource busy", the share is already mounted. Verify with:

```sh
mount | grep samba-storage
```

Confirm files are accessible:

```sh
ls /mnt/samba-storage/YouTube-downloads/
```

Expected folders: `Awesome Open Source`, `Matt Hodge Music`, `Piano with Melanie`, `Pindex`, `Tailscale`, `The Keys Coach`, `The Weavers of Eternity Paracord`

---

## Step 10 — Move Media to Samba Storage (2TB Drive)

Rather than storing movies on jellyfin-vm's local disk, all media is stored on the 2TB drive attached to samba-storage-vm ([SAMBA-STORAGE-VM-IP]), mounted at `/mnt/storage`. This keeps media centralized and separate from the Jellyfin VM.

On samba-storage-vm, create the movies folder:

```sh
sudo mkdir -p /mnt/storage/movies
```

Set ownership so the `mark` user can write to it:

```sh
sudo chown mark:mark /mnt/storage/movies
```

On jellyfin-vm, move all movies from local storage to the Samba share:

```sh
sudo mv /srv/jellyfin/movies/*.mkv /mnt/samba-storage/movies/
```

Verify the files are there:

```sh
ls /mnt/samba-storage/movies/
```

---

## Step 11 — Update Jellyfin Movies Library Path

After moving media to the Samba share, update the Jellyfin Movies library to point to the new location.

Jellyfin does not allow editing a folder path directly. Delete the existing Movies library and recreate it:

1. Dashboard → **Libraries** → **Delete** the Movies library
2. **Add Media Library**
3. Content type: **Movies**
4. Folder path: `/mnt/samba-storage/movies`
5. Click **OK** → **Save**

**Important:** After changing a library path, Jellyfin requires a rescan before playback works. Wait for the scan to complete before attempting to play any files. The scan runs automatically after saving — watch for it to finish in the dashboard before testing playback.

---

## Step 12 — Reboot Test

Reboot jellyfin-vm to confirm the Jellyfin service and Samba mount both survive a restart:

```sh
sudo reboot
```

After the VM comes back up, verify the service is running:

```sh
sudo systemctl status jellyfin
```

Look for `Active: active (running)`. Key things to confirm in the output:
- `enabled` — service is set to start at boot
- `Core startup complete` — clean startup with no errors
- `Startup complete` in under 15 seconds — normal startup time

Verify the Samba mount came back up automatically:

```sh
mount | grep samba-storage
```

Confirm media is accessible:

```sh
ls /mnt/samba-storage/movies/
```

Then open the Jellyfin web UI at `http://[JELLYFIN-VM-IP]:8096` and confirm playback works.

---

## Current State

- Jellyfin 10.11.10 installed and running ✅
- Service enabled and survives reboot ✅
- Movies library configured at `/mnt/samba-storage/movies` (on 2TB drive) ✅
- YouTube downloads library configured at `/mnt/samba-storage/YouTube-downloads` ✅
- Samba mount persistent via `/etc/fstab` and survives reboot ✅
- Playback confirmed in browser and on iPhone via Tailscale ✅
- All media stored on samba-storage-vm 2TB drive ✅

## Copying Movies to the Server (Standard Method)

From lpt-HP, use `scp` to copy movie files directly to the 2TB drive on samba-storage-vm:

```sh
scp "/home/mark/Downloads/Movie Name.mkv" mark@[SAMBA-STORAGE-VM-IP]:/mnt/storage/movies/
```

- Quotes are required if the filename contains spaces
- The file lands directly in `/mnt/storage/movies/` on the 2TB drive
- Jellyfin picks it up automatically on the next library scan

---

## Immich Integration — Attempted and Abandoned

An attempt was made to connect Jellyfin to Immich's photo library. The following was set up and then fully rolled back.

**What was tried:**
- NFS export configured on immich-vm at `/mnt/immich-storage`
- UFW rules added on immich-vm for ports 111 and 2049 from jellyfin-vm ([JELLYFIN-VM-IP])
- fstab entry added on jellyfin-vm to mount via NFS at `/mnt/immich-media`
- Jellyfin Photos library pointed at Immich's UUID-based upload folder

**Why it was abandoned:**
Immich stores media in a deeply nested UUID folder structure (`/mnt/immich-storage/upload/<uuid>/00/00/filename.jpg`). While Jellyfin could index the files, the browsing experience was unusable — no meaningful organization, no album context, no facial recognition data.

**Full rollback performed:**
- Immich Photos library deleted from Jellyfin
- NFS unmounted on jellyfin-vm (`sudo umount /mnt/immich-media`)
- fstab entry removed on jellyfin-vm
- UFW rules for ports 111 and 2049 removed on immich-vm
- immich-vm UFW restored to original state (SSH, port 2283, SNMP only)

**Future plan for photos in Jellyfin:**
Once Immich albums are fully organized, download albums as zip files, unzip and organize into a folder structure by year/person on samba-storage-vm, then add as a Jellyfin Photos library. This approach gives clean, browsable organization on the TV without touching Immich's internal storage.

---

## Current State

- Jellyfin 10.11.10 installed and running ✅
- Service enabled and survives reboot ✅
- Movies library at `/mnt/samba-storage/movies` (2TB drive) ✅
- Home Videos library at `/mnt/samba-storage/YouTube-downloads` (2TB drive) ✅
- Samba mount persistent via `/etc/fstab` and survives reboot ✅
- Playback confirmed in browser, iPhone via Tailscale, and TV ✅
- All media stored on samba-storage-vm 2TB drive ✅
- Immich integration not in place — see future plan above

## Pending

- When Immich albums are organized: download zips, unzip, organize by year/person on samba-storage-vm, add as Jellyfin Photos library
