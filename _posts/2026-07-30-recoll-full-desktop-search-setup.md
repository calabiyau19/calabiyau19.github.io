---
layout: post
draft: false
title: Install Recoll desktop search app and run first index
date: 2026-07-30
description: "This post documents the steps to set up Recoll desktop search on Linux Mint.  It also documents how to set up the initial index (exlcudes and includes) and run the first searh on that indexing"

---



## Setting Up System-Wide File Search on lpt-HP with Recoll

<!-- Hero image suggestion: a simple illustration of a magnifying glass over a stylized file tree / folder structure, in the site's dark/teal color scheme -->

I wanted a real full-text search tool on lpt-HP (my Linux Mint management laptop) — something that could search file *contents*, not just filenames, across my home directory, an external backup drive, and a couple of shares from other machines on the network. I ended up with [Recoll](https://www.recoll.org/), a mature Xapian-based search tool. Getting the scope right took a few wrong turns, so I'm documenting what actually happened, not just the clean version.

### Why Recoll, not a filename-search tool

I initially looked at a tool called EverythingX, but it turned out to only index *filenames*, not file contents — no good for searching inside my Obsidian vault notes. Recoll does real full-text indexing and has been maintained for roughly 20 years, built on the Xapian search library.

### Installing Recoll

Recoll's GUI package is available directly from Ubuntu's universe repo (Linux Mint tracks Ubuntu's base, so this works without adding any PPA):

```sh
sudo apt install recoll
```

This pulls in `recoll` (the GUI), `recollcmd`, `recollgui`, and the Qt dependencies.

### Deciding what to actually index

The goal was "search everything on lpt-HP," but that needed real scoping decisions:

- **User-only, not root** — Recoll runs as your own user account, so anything owned by root or other users is silently skipped. That's an accepted limitation, not a bug.
- **Local drives already mounted under `/mnt`** — a Samba share (`network-storage`, containing movies/music) and several drives from my Windows machine (`win_e`, `win_f`, `win_g`, `win_c`) were already visible locally, so no extra SSHFS/NFS setup was needed for those.
- **Remote servers (NUC5/NUC3 VMs)** — Recoll has no native network protocol. It can only index local filesystem paths. To search a remote server, its filesystem would need to be mounted locally first (SSHFS/NFS/Samba), then added to the config like any local folder. I decided to treat this as a future phase rather than bite it off immediately.

### The config file

Recoll's per-index settings live in `~/.recoll/recoll.conf`, auto-created on first launch.

```sh
recoll
```

Cancel the first-run auto-index prompt if it appears, so the config can be set up before the first real indexing pass.

```sh
nano ~/.recoll/recoll.conf
```

**Final `topdirs`** — the paths to actually walk:

```
topdirs = / /media/mark/Ext_T7_lpthp /mnt/win_c/Users/marks/Documents /mnt/win_c/Users/marks/Downloads
```

Setting `topdirs` to `/` covers the whole local filesystem, including everything already mounted under `/mnt` (`network-storage`, `win_e`, `win_f`, `win_g`). `win_c` is a full Windows C: drive — Program Files, Windows folder, etc. — so only two specific subfolders (Documents, Downloads) were added explicitly rather than the whole share.

**Final `skippedPaths`** — directories to never walk into:

```
skippedPaths = /proc /sys /dev /run /tmp /var/tmp /usr /var /bin /sbin /lib /lib64 /boot /snap /opt ~/.cache ~/.local/share/Trash /mnt/win_c /mnt/win_f /mnt/win_g /media/mark/Ext_T7_lpthp/timeshift /media/mark/Ext_T7_lpthp/system-backups /media/mark/Ext_T7_lpthp/dejadup-backups /media/mark/Ext_T7_lpthp/home-backup-20260611 "/media/mark/Ext_T7_lpthp/Ghost Backup" /media/mark/Ext_T7_lpthp/email-backups /media/mark/Ext_T7_lpthp/Browser-Backups /media/mark/Ext_T7_lpthp/Shell-History-Backup
```

**Final `skippedNames`** — filename/pattern matches skipped anywhere in the tree:

```
skippedNames = .git node_modules __pycache__ .venv venv $RECYCLE.BIN .Trash .Trash-1000 lost+found *.qcow2 *.vmdk *.iso *.img .cargo
```

A few notes on why these exclusions exist:

- **`/usr /var /bin /sbin /lib* /boot /opt /snap`** — pure OS and program internals per the Linux Filesystem Hierarchy Standard, never personal files. Confirmed by running `sudo du -x --max-depth=3 -h / | sort -rh | head -50` to see what was actually consuming space before excluding it.
- **`~/.cache`** — browser/app caches, no search value, can be several GB of pure noise.
- **`/mnt/win_c`** (except the two named subfolders) — a full Windows system drive. Note: simply *not listing* the whole share in `topdirs` does **not** exclude it if `/` is also in `topdirs` — `skippedPaths` is the only mechanism that actually blocks the walk. I got this wrong on the first attempt and let the whole C: drive get indexed for 12+ hours before catching it.
- **`/mnt/win_f`** — this drive contains a ~50,000-photo dump that was uploaded to Immich and is now kept only as cold-storage backup; redundant with Immich's own indexing.
- **`/mnt/win_g`** — cold-storage backups including Windows `$RECYCLE.BIN` and `.Trash` folders.
- **Ext_T7 backup folders** (`timeshift`, `system-backups`, `dejadup-backups`, dated home backup, Ghost/email/browser/shell-history backups) — full snapshot/backup archives, not searchable content. `Website-Backups` was deliberately kept searchable.
- **`.git`, `node_modules`, `__pycache__`, `.venv`/`venv`, `.cargo`** — package manager caches and VCS internals, third-party downloaded content rather than authored files.
- **`$RECYCLE.BIN`, `.Trash`, `.Trash-1000`, `lost+found`** — deleted-file history and filesystem recovery junk.
- **`*.qcow2`, `*.vmdk`, `*.iso`, `*.img`** — VM disk images and ISOs, large binary blobs with no text content to index.

### Running the index

```sh
recollindex
```

The first (badly-scoped) attempt ran for over 12 hours and hit 91GB before being stopped — almost entirely due to the `win_c` full-drive mistake above. After correcting the config and wiping the partial index:

```sh
rm -rf ~/.recoll/xapiandb
```

```sh
recollindex
```

The corrected run completed in about 50 minutes, producing a 6.0GB index — a reasonable size for the actual intended scope.

### Fixing the tiny UI text

Recoll's Qt-based interface opened unreadably small. Recoll does have a **Display scale** setting under Preferences → User interface, but the dialog itself was too small to comfortably use at first. The simplest fix: temporarily drop the monitor's display scaling from 125% to 100%, which made the Recoll preferences dialog usable long enough to set **Display scale to 2.70** — after that, the setting persists on its own and the monitor scaling can go back to normal.

### Setting up a keybinding

Cinnamon → Keyboard → Shortcuts → find an unused binding (I reused `Super+S`, previously mapped to "Show Desklets," a feature I never use) → assign it to the command `recoll`.

### Verifying it actually works

```sh
recoll
```

A search for a known term (a song title from an Obsidian note) returned the correct result — confirming the index is genuinely searching file contents, not just filenames.
