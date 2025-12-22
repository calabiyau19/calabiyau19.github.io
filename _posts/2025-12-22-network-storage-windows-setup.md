---
layout: post
draft: false
title: "Windows Access to Samba Network Storage"
date: 2025-12-22
description: "This guide documents how to configure Windows machines to access the Samba network storage server"
categories: [networking, samba, windows]
---

## Windows Access to Samba Network Storage

This guide documents how to configure Windows machines to access the Samba network storage server running on VM 1106.

## Prerequisites

- Samba storage VM running at IP: `192.168.1.167`
- Samba share name: `storage`
- Samba username: `mark`
- Samba password: [your configured password]
- Windows machine on same local network (or connected via proxmox)

## Configuration Steps

### Step 1: Open File Explorer

- Press `Windows key + E` to open File Explorer
- Or click the File Explorer icon in the taskbar

### Step 2: Access Map Network Drive Dialog

- In File Explorer, right-click on "This PC" in the left sidebar
- Select "Map network drive..." from the context menu

### Step 3: Configure Drive Mapping

In the Map Network Drive dialog:

- **Drive letter**: Select a drive letter from the dropdown (Z: is commonly used for network drives)
- **Folder**: Enter the following path:
```sh
\\192.168.1.167\storage
```

- ✅ **Check**: "Reconnect at sign-in" (ensures drive remounts after reboot)
- ✅ **Check**: "Connect using different credentials" (allows specifying Samba username)
- Click the **"Finish"** button

### Step 4: Enter Samba Credentials

When the Windows Security credential prompt appears:

- **Username**: Enter `mark`
- **Password**: Enter the Samba password configured on VM 1106
- ✅ **Check**: "Remember my credentials" (stores credentials in Windows Credential Manager)
- Click **"OK"**

### Step 5: Verify Access

- The mapped network drive should appear in File Explorer under "This PC"
- The drive will show as "storage (\\192.168.1.167) (Z:)" or similar
- You should be able to browse, read, and write files to this location
- Test by creating a test file or folder to confirm write access

## Per-User Configuration

**Important**: Network drive mappings in Windows are **per-user account**.

- Each Windows user account must map the drive independently
- Each user follows the same steps above under their own Windows login
- All users authenticate with the same Samba username (`mark`) and password
- Credentials are stored separately for each Windows user in their own Credential Manager

## Network Access Methods

### Local Network Access

Use the local IP address:
```sh
\\192.168.1.167\storage
```

- Works when Windows machine is on same LAN as VM 1106
- This is the recommended method for local access

### Remote Access via Tailscale

Use the Tailscale IP address:
```sh
\\[tailscale-ip]\storage
```

- Replace `[tailscale-ip]` with the Tailscale IP assigned to VM 1106
- Works from anywhere when both machines are connected to Tailscale network
- Check Tailscale admin console or run `tailscale ip` on VM 1106 to find the IP

## Troubleshooting

### Drive Doesn't Reconnect After Reboot

1. Verify "Reconnect at sign-in" was checked during setup
2. Check Windows Credential Manager to ensure credentials are saved:
   - Control Panel → User Accounts → Credential Manager → Windows Credentials
   - Look for entry: `192.168.1.167`
3. If credentials are missing, re-map the drive following the steps above

### Access Denied Errors

1. Credentials may have been cleared or expired
2. Re-map the drive following the configuration steps
3. Ensure Samba password hasn't changed on VM 1106
4. Verify the Samba user `mark` has appropriate permissions

### Can't Connect to \\192.168.1.167

1. Verify VM 1106 is running on Proxmox host .125:
```sh
# On Proxmox host .125
qm status 1106
```

2. Ping the VM from Windows Command Prompt:
```sh
ping 192.168.1.167
```

3. Verify Samba service is running on VM 1106:
```sh
# On VM 1106
sudo systemctl status smbd
```

4. Check firewall rules on VM 1106:
```sh
# On VM 1106
sudo ufw status
```

Ensure ports 139 and 445 are allowed for Samba.

### Credential Prompt Keeps Appearing

1. Delete existing credentials from Windows Credential Manager
2. Re-map the drive ensuring "Remember my credentials" is checked
3. Verify the Samba password is correct

## Samba Server Configuration Details

### VM Information

- **VM ID**: 1106 on Proxmox host .125 (nuc5)
- **VM IP**: 192.168.1.167
- **Share name**: storage
- **Share path on VM**: `/mnt/storage`
- **Storage device**: 2 TB Samsung T7 external SSD
- **Valid user**: mark
- **Permissions**: read/write (0775)

### Related Services

- **Samba service**: `smbd`
- **Tailscale**: Installed and configured for remote access
- **Auto-mount**: External SSD auto-mounts at `/mnt/storage` on VM boot

### Configuration Files on VM 1106

Main Samba configuration:
```sh
/etc/samba/smb.conf
```

Auto-mount configuration for external SSD:
```sh
/etc/fstab
```

## Summary

This guide covered configuring Windows machines to access the centralized Samba network storage server. The setup involves mapping a network drive using the VM's IP address and storing credentials in Windows Credential Manager for persistent access. Each Windows user account requires independent configuration, but all authenticate with the same Samba credentials. The mapped drive automatically reconnects on reboot and provides transparent access to the shared storage as if it were a local drive.

The storage is accessible both locally via `\\192.168.1.167\storage` and remotely via Tailscale using the VM's Tailscale IP address. This provides flexible access to 1.7 TB of shared storage from any Windows machine on the network or connected via Tailscale VPN.

**Tested Configuration**: Windows 10/11 with user accounts mark and mary  
**Date Configured**: December 22, 2024  
**Status**: ✅ Working - drive auto-reconnects on reboot, credentials persistent across sessions