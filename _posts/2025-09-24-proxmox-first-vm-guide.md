---
title: Creating Your First VM on Proxmox - Part 1
date: 2025-09-23
description: "Step-by-step guide to create your first virtual machine on Proxmox using Ubuntu 24.04 Server. Covers downloading the ISO, uploading to Proxmox, VM creation, and basic Ubuntu installation with SSH enabled."
---

## Creating Your First VM on Proxmox - Part 1

### Overview

Now that you have Proxmox installed and configured, it's time to create your first virtual machine (VM). This guide will walk you through creating a Ubuntu 24.04 Server VM using the Proxmox web interface. We'll keep things simple and use mostly default settings to get you started.

This is Part 1 of a series - we'll create the VM and install Ubuntu Server with SSH enabled, then in Part 2 we'll install applications like speedtest-tracker.

### Step-by-Step VM Creation

#### Step 1: Download Ubuntu 24.04 Server ISO

First, download the Ubuntu Server ISO to your local computer:

1. **Visit the Ubuntu Server download page**: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)
2. **Download Ubuntu 24.04 LTS Server**: Click "Download Ubuntu Server 24.04 LTS"
3. **Save the file**: The ISO will be named something like `ubuntu-24.04-live-server-amd64.iso`
4. **Note the file location**: Remember where you saved it (usually Downloads folder)

> **Note**: The download is about 2.5GB, so it may take a few minutes depending on your internet speed.

#### Step 2: Understanding the Proxmox Interface

Before we upload the ISO, let's get familiar with the Proxmox web interface so you know what you're looking at:

**First Login Orientation:**

1. **Open Proxmox web interface**: Go to `https://192.168.1.121:8006` (or your Proxmox IP)
2. **Log in** with your root credentials

**Understanding the Interface Layout:**

The Proxmox interface has four main regions:

**Resource Tree (Left Side):**
- This is your main navigation area
- Shows the **Server View** by default, which displays:
  - **Datacenter**: Contains cluster-wide settings
  - **Your Node**: This is your Proxmox server (named whatever you chose during installation, e.g., "sv-proxmox")
  - **Storage**: Data storage locations under your node
  - **Guest**: VMs and containers (will appear here after we create them)

**Content Panel (Center Area):**
- Changes based on what you select in the Resource Tree
- Shows tabs and options relevant to your selection
- This is where you'll find tabs like "Summary", "ISO Images", "Content", etc.

**Main Display (Right Side):**
- Shows detailed information based on what you click in the Content Panel
- This is where file lists, settings, and detailed views appear

**Navigation Flow for Beginners:**
The key to understanding Proxmox navigation:
**Resource Tree → Content Panel → Main Display**

When you click something in the Resource Tree (left), the Content Panel (center) changes to show relevant options. When you click a tab in the Content Panel, the Main Display (right) shows that content.

**Practice Navigation:**
1. **Click "Datacenter"** in Resource Tree → Content Panel shows datacenter options
2. **Click your node name** (e.g., "sv-proxmox") in Resource Tree → Content Panel shows node options  
3. **Expand your node** by clicking the arrow next to it → Shows storage and other items underneath
4. **Click "local (your-node-name)"** → Content Panel shows storage-related tabs including "ISO Images"

Now you understand how to navigate to upload the ISO!

#### Step 3: Upload ISO to Proxmox

Now we need to upload the ISO from your Linux Mint laptop to your Proxmox server:

**Important: Understanding the Upload Process**
- You downloaded the ISO to your Linux Mint laptop (probably in Downloads folder)
- The Proxmox web interface is running in your browser on that same laptop
- When you click "Select File...", you're browsing YOUR laptop's files
- The "Upload" button transfers the file from your laptop TO the Proxmox server over your network

**Upload Steps:**

1. **Navigate to ISO storage**:
   - In the left panel, click on **"local (sv-proxmox)"** (your local storage)
   - In the right panel, click on the **"ISO Images"** tab

2. **Start the upload**:
   - Click the **"Upload"** button (should be near the top of the right panel)
   - A dialog box will appear

3. **Select your local ISO file**:
   - Click **"Select File..."**
   - **This opens YOUR Linux Mint file browser** (not the Proxmox server)
   - Navigate to your Downloads folder: `/home/yourusername/Downloads/`
   - Select `ubuntu-24.04-live-server-amd64.iso`
   - Click **"Open"**

4. **Upload the file**:
   - Back in the Proxmox upload dialog, click **"Upload"**
   - **Progress bar will show**: The file is being transferred from your laptop to Proxmox
   - **Wait patiently**: This takes 3-10 minutes depending on your network speed
   - **Don't close your browser** during the upload

5. **Verify upload completed**:
   - When finished, you'll see the ISO listed in the "ISO Images" tab
   - The file is now stored on your Proxmox server, ready to use

> **Tip**: The upload transfers the entire 2.5GB file over your home network. If it seems slow, that's normal - just be patient and don't interrupt it.

#### Step 4: Create the Virtual Machine

Now we'll create the actual VM:

1. **Start VM creation**:
   - Click **"Create VM"** button (top right)
   - The "Create: Virtual Machine" wizard will open

2. **General tab**:
   - **Node**: Leave as default (your Proxmox server)
   - **VM ID**: Leave as default (usually 100 for first VM)
   - **Name**: Enter a name like `ubuntu-server-01`
   - Click **"Next"**

3. **OS tab**:
   - **Use CD/DVD disc image file (iso)**: Select this option
   - **Storage**: local
   - **ISO image**: Select your uploaded `ubuntu-24.04-live-server-amd64.iso`
   - **Guest OS**: 
     - Type: **Linux**
     - Version: **6.x - 2.6 Kernel**
   - Click **"Next"**

4. **System tab**:
   - **Graphics card**: Default (VGA)
   - **Machine**: Leave as default (q35)
   - **BIOS**: Leave as default (SeaBIOS)
   - **SCSI Controller**: Leave as default (VirtIO SCSI single)
   - **Qemu Agent**: **✓ Check this box** (enables better VM management)
   - Click **"Next"**

5. **Disks tab**:
   - **Storage**: local-lvm
   - **Disk size (GiB)**: **32** (good starting size for Ubuntu Server)
   - **Format**: Raw
   - **Cache**: Default (No cache)
   - Leave other settings as default
   - Click **"Next"**

6. **CPU tab**:
   - **Sockets**: 1
   - **Cores**: **2** (good starting point)
   - **Type**: Default (kvm64)
   - Click **"Next"**

7. **Memory tab**:
   - **Memory (MiB)**: **2048** (2GB RAM - good starting point)
   - **Minimum memory**: Leave empty
   - **Ballooning Device**: ✓ Checked
   - Click **"Next"**

8. **Network tab**:
   - **Bridge**: vmbr0 (default)
   - **Model**: VirtIO (paravirtualized)
   - **MAC address**: Leave as auto-generated
   - **Firewall**: ✓ Checked
   - Click **"Next"**

9. **Confirm tab**:
   - Review your settings
   - **Start after created**: ✓ Check this box
   - Click **"Finish"**

#### Step 5: Install Ubuntu Server

Your VM will automatically start after creation. Now we'll install Ubuntu:

1. **Access the VM console**:
   - In the left panel, click on your VM (e.g., "100 (ubuntu-server-01)")
   - Click **"Console"** tab
   - You should see the Ubuntu installer loading

2. **Ubuntu Installation Process**:

   **Language Selection**:
   - Select **"English"** (or your preferred language)
   - Press Enter

   **Keyboard Configuration**:
   - Leave as **"English (US)"** or select your keyboard layout
   - Press Enter to continue

   **Network Connections**:
   - **✅ IMPORTANT**: Note the IP address shown (e.g., `192.168.1.xxx/24`)
   - **Write this IP address down** - you'll need it for SSH later
   - Leave network settings as default (DHCP)
   - Press Enter to continue

   **Configure Proxy**:
   - Leave blank unless you use a proxy
   - Press Enter to continue

   **Configure Ubuntu Archive Mirror**:
   - Leave as default
   - Press Enter to continue

   **Guided Storage Configuration**:
   - Select **"Use an entire disk"**
   - Select your virtual disk (should be the only option)
   - Press Enter to continue

   **Storage Configuration**:
   - Review the disk layout (should use full 32GB)
   - Select **"Done"**
   - **Confirm destructive action**: Select **"Continue"**

   **Profile Setup** (✅ Important):
   - **Your name**: Enter your real name (e.g., "John Smith")
   - **Your server's name**: Enter a hostname (e.g., "ubuntu-vm")
   - **Pick a username**: Choose a username (e.g., "admin" or your preferred name)
   - **Choose a password**: Create a strong password
   - **Confirm your password**: Re-enter the same password
   - **✅ Write down these credentials** - you'll need them!
   - Press Enter to continue

   **SSH Setup** (✅ Critical):
   - **Install OpenSSH server**: **✓ CHECK THIS BOX**
   - **Import SSH identity**: Leave as "No" unless you have SSH keys
   - Press Enter to continue

   **Featured Server Snaps**:
   - Leave all unchecked for now (we'll install what we need later)
   - Press Enter to continue

3. **Wait for Installation**:
   - Ubuntu will now install (takes 5-10 minutes)
   - You'll see package installation progress
   - **Do not close the console window**

4. **Installation Complete**:
   - When you see **"Install complete!"**
   - Select **"Reboot Now"**
   - The VM will reboot

#### Step 6: First Boot and SSH Access

After reboot, you'll have a running Ubuntu Server:

1. **Wait for boot to complete**:
   - You'll see the Ubuntu login prompt in the console
   - Login with the username/password you created

2. **Check the IP address**:
   - Run: `ip addr show`
   - Note the IP address (should be the same as during installation)
   - Example output: `inet 192.168.1.150/24`

3. **Test SSH from your main computer**:
   ```bash
   ssh admin@192.168.1.150
   # Replace 'admin' with your username and use the correct IP
   # Enter your password when prompted
   ```

#### Step 7: Post-Installation Updates

Once logged in via SSH, perform essential updates:

```bash
# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install common useful packages
sudo apt install -y curl wget htop tree

# Reboot if kernel was updated
sudo reboot
```

After reboot, SSH back in to confirm everything is working.

### What You've Accomplished

Congratulations! You now have:

- ✅ A fully functional Ubuntu 24.04 Server VM running on Proxmox
- ✅ SSH access enabled for remote administration
- ✅ System fully updated with latest packages
- ✅ Basic useful tools installed
- ✅ Network connectivity working

### Next Steps

In **Part 2**, we'll install Docker and set up speedtest-tracker to monitor your internet connection speeds.

### Quick Reference

**VM Details Created**:
- **Name**: ubuntu-server-01
- **OS**: Ubuntu 24.04 LTS Server
- **RAM**: 2GB
- **CPU**: 2 cores
- **Disk**: 32GB
- **Network**: Bridged to your home network

**Important Credentials to Save**:
- **Proxmox Web**: `https://192.168.1.121:8006` (root + password)
- **VM SSH**: `ssh username@VM_IP_ADDRESS`
- **VM Console**: Available through Proxmox web interface

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Can't SSH to VM | Check IP address with `ip addr show`, ensure SSH was enabled during install |
| VM won't start | Check Proxmox logs, ensure adequate RAM/storage available |
| Console is black | Try pressing Enter or clicking in console window |
| Network not working | Check VM network settings match your home network setup |
| Forgot VM password | Reset through VM console, create new user with sudo access |

### Additional Resources

- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Proxmox VE Administration Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.html)
- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)