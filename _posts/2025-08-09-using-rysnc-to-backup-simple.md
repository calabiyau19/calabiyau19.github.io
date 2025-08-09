---
layout: post
title: "Using rsync to back up websites or folders across the network very simply"
draft: false
date:  2025-08-09
description: Rsync is a simple but powerful and fast way to backup files and folders across the network.  In this case, I an backing up my ADGuard instillation running on one linux box to a backup partition on another linux box and it is extremely fast.  
---

Step 1 — Make the destination folder on your backup drive
On your local machine:

```sh
mkdir -p "/media/Linux_Extra/Website-Backups/AdGuard"
```

Step 2 — Change permissions on lpt-dell so you can read the files
On lpt-dell:

```sh
sudo chown -R mark:mark /home/mark/adguard-home
```

This makes sure you own all AdGuard’s files, so rsync can read them.

Step 3 — Run rsync backup
On your local machine:

```sh
rsync -av mark@lpt-dell:/home/mark/adguard-home/ /media/Linux_Extra/Website-Backups/AdGuard/
```

That’s it.
If this works, we can later make it fancier (add sudo, include multiple backups, etc.), but this basic version shows how rsync works and how easy it is to use.


How to install Rsync.

Check if installed first.

```sh
rsync --version
```
If not installed:

```sh
sudo apt update
sudo apt install rsync
```
Once rsync is installed on both ends, your commands like:

```sh
rsync -av user@source:/path/to/files /path/to/destination
```

will work as long as SSH access is set up.
