---
layout: post
title: "Proxmox - Debian Maintenance Guide"
draft: false
date: 2025-09-15
description: Maintenance Guide for Debian based Proxmox servers.  Differs a little from the same tasks on Ubuntu based servers.  Don't make the mistake I did of running Ubuntu maintenance script on Debian servers, in this case they were Proxmox ones.  
---


## Complete Process: Create and Run `proxmox-maint` from Anywhere

Works on Proxmox VE 7/8 or Debian 11/12

### Step 1: Create the `/root/Scripts/` Directory

If the directory doesn't already exist, create it:

```sh
mkdir -p /root/Scripts
```

### Step 2: Create the Script File with `nano`

Open nano to create the script file:

```sh
nano /root/Scripts/proxmox-maint.sh
```

### Step 3: Paste the Script into Nano

Paste this complete script inside the editor:

```sh
#!/bin/sh
set -e

echo "Updating APT cache..."
apt update

echo "⬆Running dist-upgrade..."
apt dist-upgrade -y

echo "Cleaning up unused packages and cache..."
apt autoremove -y
apt autoclean
apt clean

echo "Purging old config packages..."
dpkg -l | awk '/^rc/ { print $2 }' | xargs -r apt purge -y

echo "Vacuuming old journal logs (7d)..."
journalctl --vacuum-time=7d

echo "Checking disk usage..."
df -h

echo "Checking ZFS pool status (if installed)..."
if command -v zpool &>/dev/null; then
  zpool status
  zfs list
else
  echo "ZFS not installed or in use."
fi

echo "Proxmox maintenance complete."
```

#### Save and Exit `nano`

- Press `CTRL+O`, then `Enter` to save
- Press `CTRL+X` to exit

### Step 4: Make the Script Executable

Make the script executable with the following command:

```sh
chmod +x /root/Scripts/proxmox-maint.sh
```

You can test it manually at this point by running:

```sh
/root/Scripts/proxmox-maint.sh
```

### Step 5: Create a Symlink to Make It Globally Executable

Create a symbolic link to make the script accessible from anywhere:

```sh
ln -s /root/Scripts/proxmox-maint.sh /usr/local/bin/proxmox-maint
```

Now you can run the script from **anywhere** like this:

```sh
proxmox-maint
```

No need to use `./` or give the full path.

### Step 6: (Optional) Confirm It Works from Anywhere

Try running the script from your home directory to verify it works:

```sh
cd ~
proxmox-maint
```

### Step 7: (Optional) View Script Location with `which`

Verify the symlink is working correctly:

```sh
which proxmox-maint
```

Should output:

```
/usr/local/bin/proxmox-maint
```

## You're Done!

You now have:

- A reusable, version-controlled script in `/root/Scripts/`
- Executable from anywhere via `proxmox-maint`
- Clean output showing each maintenance step
- Safe for use on Proxmox (Debian-based) systems