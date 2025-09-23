---
title: Proxmox Installation Guide on a mini pc 
date: 2025-09-23
description: "This article documents the general steps I followed to install Proxmox servers on two mini PCs, where I replaced Windows 11 Pro with Proxmox.  For a total beginner, my advice would be to feed this into Claude or ChatGPT and have them walk you through it step by step.  But, read it first and try to understand as much as possible, as Claude and ChatGPT can and will make assumptions that may turn out not to be true for your installation."
---

## Proxmox VE Installation Guide

### Overview

Proxmox VE (Virtual Environment) is a powerful, open-source virtualization platform based on Debian Linux. It's designed to host virtual machines (VMs), containers, and more.

Your mini PC may have secure boot, UEFI, NVMe drives, or non-standard BIOS defaults — we'll handle those issues step-by-step.

### Step-by-Step Installation Guide

#### Step 1: Prepare a USB Installer

##### 1. Download the Latest Proxmox ISO

Visit the official Proxmox download page: [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)

Download the Proxmox VE ISO Installer:
- **Latest version**: [Proxmox VE 9.0 ISO Installer](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso/proxmox-ve-9-0-iso-installer)
- **Last stable version**: [Proxmox VE 8.4 ISO Installer](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso/proxmox-ve-8-4-iso-installer)

##### 2. Create a Bootable USB

**Option A: Using Rufus (Windows)**

1. Download Rufus: [https://rufus.ie/](https://rufus.ie/)
2. Insert your USB drive (at least 4 GB recommended)
3. Open Rufus and configure:
   - **Device**: your USB stick
   - **Boot selection**: select the downloaded Proxmox ISO
   - **Partition scheme**: GPT
   - **Target system**: UEFI (non-CSM)
   - **File system**: FAT32
4. Click **Start** — Rufus will wipe the USB and write Proxmox

**Option B: Using Balena Etcher (Windows/macOS/Linux)**

1. Download Balena Etcher: [https://etcher.balena.io/](https://etcher.balena.io/)
2. Insert your USB drive (at least 4 GB recommended)
3. Open Balena Etcher:
   - **Flash from file**: select the downloaded Proxmox ISO
   - **Select target**: choose your USB drive
   - **Flash**: start the flashing process

> **Warning**: All data on the USB drive will be erased.

#### Step 2: Boot Mini PC from USB

##### 1. Access BIOS/UEFI Settings

Reboot the mini PC and immediately press the BIOS key:
- Common keys: `Del`, `F2`, `F7`, or `Esc`
- Some mini PCs show this at boot as: "Press F2 to enter Setup"

##### 2. Configure BIOS Settings

These settings vary by vendor — try to match these as closely as possible:

- **Secure Boot** → **Disabled**
- **UEFI Boot Mode** → **Enabled**
- **Fast Boot** → **Disabled**
- **Virtualization (Intel VT-x / AMD-V)** → **Enabled**
- **Boot Order** → **Set USB first**

Save and Exit.

#### Step 3: Install Proxmox VE

##### 1. Boot into USB Installer

After BIOS setup, your mini PC should boot into the Proxmox Installer menu.

If not, reboot and press the boot menu key (`F7`, `F12`, etc.) and manually choose the USB drive.

##### 2. Start Installation

Choose: **Install Proxmox VE** (NOT Debug or Advanced)

The installer will load and begin setup.

> **Note**: If you encounter disk detection issues, see the [Troubleshooting section](#troubleshooting-common-issues) below.

##### 3. Handle Common Error: "No Valid Hard Disk Found"

If Proxmox doesn't see your NVMe/SSD:

1. Go back into BIOS
2. Check that **SATA Mode** is set to **AHCI**
3. Check if **RAID Mode** is enabled — **disable RAID**
4. If you see **Intel VMD**, **disable it** too
5. Reboot the installer — now your disk should be visible

##### 4. Installation Steps

**Accept License Agreement**
- Click "I Agree"

**Select Target Disk**
- Pick the main SSD (e.g., `/dev/nvme0n1`)
- Click **Options** if you want to adjust:
  - File System (defaults to ext4, which is fine)
  - Swap size, LVM options, etc. (leave default for now)

**Set Location and Keyboard**
- Pick your region, language, and keyboard layout

**Set Root Password and Email**
- This is the admin password for the Proxmox server
- Use a strong password
- Email is for alerts (optional, but recommended)

**Network Setup**
- Pick the main Ethernet interface (e.g., `enp2s0`)
- Set the hostname (e.g., `sv-proxmox`)
- Set a static IP address, subnet mask, gateway, and DNS

Example configuration:
- **IP**: `192.168.1.121`
- **Subnet**: `255.255.255.0`
- **Gateway**: `192.168.1.1`
- **DNS**: `1.1.1.1` or `8.8.8.8`

> **Important**: If you don't set a static IP, your router might change it later, and you'll lose track of the server.
> **NOTE: I did this after installation so if you miss it here you can fix it later.

##### 5. Start Installation

1. Confirm and click **Install**
2. Installation may take a few minutes
3. At the end, remove the USB and reboot

#### Step 4: First Boot and Web Access

##### 1. First Boot

After installation, the mini PC should boot to a terminal screen showing:

```
You can now connect to the Proxmox VE web interface:
https://192.168.1.121:8006
```

If you see a certificate warning in your browser — accept it or click "Proceed" (this is normal for self-signed certificates).

##### 2. Log into the Web Interface

1. Go to: `https://192.168.1.121:8006`
2. Login with:
   - **Username**: `root`
   - **Password**: (whatever you set during install)
   - **Realm**: Linux PAM

### Post-Install Configuration

#### Enable SSH Access

SSH access is essential for remote administration and running the remaining post-install commands. Here's how to set it up:

**Step 1: Enable SSH on your Proxmox server**

At your Proxmox mini PC (you should see a black terminal screen), log in and run these commands:

1. **Log in to the console** (if not already logged in):
   - Username: `root`
   - Password: (the password you set during installation)

2. **Enable and start SSH service**:
```bash
systemctl enable ssh
systemctl start ssh
```

**Step 2: Connect to Proxmox from your main computer**

Now you can remotely connect to your Proxmox server from your main computer:

**On Mac/Linux:**
- Open Terminal and run:
```bash
ssh root@192.168.1.121
# Enter your root password when prompted
```

**Step 3: Run all remaining commands via SSH**

All the commands in the following sections should be run through this SSH connection, not at the Proxmox console directly.

#### Remove Subscription Nag Popup

```bash
nano /etc/apt/sources.list.d/pve-enterprise.list
```

Comment out the line with `enterprise.proxmox.com` by adding `#` in front of it.

Then add the no-subscription repository:

```bash
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list
apt update && apt full-upgrade -y
```

#### Configure Automatic Security Updates

Set up automatic security updates for better security:

```bash
# Install unattended-upgrades
apt install unattended-upgrades -y

# Configure automatic updates
echo 'Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
    "Proxmox:${distro_codename}";
};' > /etc/apt/apt.conf.d/50unattended-upgrades

# Enable the service
systemctl enable unattended-upgrades
systemctl start unattended-upgrades
```

#### Set Up Weekly Maintenance Tasks

Create a weekly maintenance script for system updates and cleanup:

```bash
# Create maintenance script
cat > /usr/local/bin/weekly-maintenance.sh << 'EOF'
#!/bin/bash
# Weekly Proxmox maintenance script

echo "Starting weekly maintenance: $(date)"

# Update package lists
apt update

# Upgrade packages
apt full-upgrade -y

# Remove orphaned packages
apt autoremove -y

# Clean package cache
apt autoclean

# Clean old kernels (keep last 2)
pve-efiboot-tool kernel list | tail -n +3 | xargs -r pve-efiboot-tool kernel remove

echo "Weekly maintenance completed: $(date)"
EOF

# Make executable
chmod +x /usr/local/bin/weekly-maintenance.sh

# Add to crontab (runs every Sunday at 2 AM)
echo "0 2 * * 0 /usr/local/bin/weekly-maintenance.sh >> /var/log/weekly-maintenance.log 2>&1" | crontab -
```

#### Email Alert Configuration (Optional)

Note: Email alerts from Proxmox often require additional SMTP configuration to work reliably. Many users report not receiving email notifications without proper mail server setup.

If you need email alerts, consider configuring an external SMTP server:

```bash
# Install postfix for email relay
apt install postfix -y
# Configure according to your email provider's SMTP settings
```

### Congratulations!

You now have a fully installed Proxmox server ready to:

- Create and run VMs
- Add storage (LVM, ZFS, NFS, etc.)
- Use the web GUI to manage everything
- Expand into clustering, backups, automation, and more

### Troubleshooting Common Issues

| Issue | Fix |
|-------|-----|
| USB not booting | Use Rufus (Windows) or Balena Etcher (cross-platform) with GPT + UEFI, disable Secure Boot |
| "No hard disk found" | Disable Intel VMD / RAID in BIOS, set SATA mode to AHCI |
| Can't access Proxmox web UI | Make sure IP is correct, check router, use `ip addr` command |
| Certificate warning | Accept it — it's normal for self-signed certificates |
| Mouse/keyboard don't work in BIOS | Try different USB ports, especially USB 2.0 ports |

### Additional Resources

- [Official Proxmox Documentation](https://pve.proxmox.com/pve-docs/)
- [Proxmox Community Forum](https://forum.proxmox.com/)
- [Proxmox Wiki](https://pve.proxmox.com/wiki/Main_Page)