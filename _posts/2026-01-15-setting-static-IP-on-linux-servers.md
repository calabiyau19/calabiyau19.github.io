---
layout: post
draft: false
title: "How to set a static IP address on a linux server or desktop"
date: 2026-01-15
description: "This short tutorial shows how to set a static IP address for one of your Ubuntu or Debian based servers using the command line or Linux or similar Linux desktop environments like Mint through the GUI."
---

## Setting Static IP on Linux Servers (Proxmox/Ubuntu)

### Overview
This guide shows how to configure a static IP address on a Linux server that is already using the IP you want to keep.

**CRITICAL WARNING FOR PROXMOX**: Do NOT restart networking service while VMs are running. Changes require a full server reboot.

### Prerequisites
- Root/sudo access to the server
- Current IP address you want to make static
- Gateway address (usually 192.168.1.254)
- DNS servers (recommended: 1.1.1.1 and 8.8.8.8)
- **For Proxmox**: Schedule maintenance window for server reboot

---

## For Linux Mint / Ubuntu Desktop (NetworkManager)

**RECOMMENDED: Use the GUI method**

### GUI Method (Easiest)

1. Click the network icon in your system tray (bottom right)
2. Click "Network Settings" or "Edit Connections"
3. Find your active connection (e.g., "Auto ORBI08")
4. Click the gear icon or "Edit"
5. Go to the "IPv4 Settings" tab
6. Change method from "Automatic (DHCP)" to "Manual"
7. Click "Add" and enter:
   - **Address**: `192.168.1.100` (your desired IP)
   - **Netmask**: `255.255.255.0` (or enter `24`)
   - **Gateway**: `192.168.1.254`
8. In **DNS servers** field: `1.1.1.1, 8.8.8.8`
9. Click **"Save"**
10. Disconnect and reconnect to the network

### Command Line Method (Alternative)

If you must use command line on NetworkManager systems:
```sh
# List connections to find your UUID
nmcli connection show

# Modify connection (use your actual UUID)
nmcli connection modify <UUID> ipv4.method manual ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.254 ipv4.dns "1.1.1.1 8.8.8.8"

# Apply changes
nmcli connection down <UUID> && nmcli connection up <UUID>

# Verify
ip addr show | grep "inet "
```

**Note**: NetworkManager can have duplicate connection names. If you see warnings about duplicate names, use the connection's UUID instead of the name, or use the GUI method.

---

## For Proxmox / Ubuntu Server

### Steps

#### 1. Check Current Configuration
```sh
cat /etc/network/interfaces
```

Look for your network bridge (usually `vmbr0` on Proxmox) or main interface.

#### 2. Verify if Already Static
Look for the line starting with `iface`:
- `iface vmbr0 inet static` = Already static (just add DNS if missing)
- `iface vmbr0 inet dhcp` = Using DHCP (needs conversion to static)

#### 3. Edit Network Configuration
```sh
nano /etc/network/interfaces
```

#### 4. Configure Static IP
Find or create the interface section. Example for vmbr0:
```
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.XXX/24
        gateway 192.168.1.254
        dns-nameservers 1.1.1.1 8.8.8.8
        dns-search tailcf482c.ts.net
        bridge-ports enp1s0
        bridge-stp off
        bridge-fd 0
```

Replace:
- `192.168.1.XXX` with your desired IP
- `enp1s0` with your physical interface name (check with `ip link show`)
- Remove `bridge-*` lines if not using a bridge

#### 5. Save and Exit
- Press `Ctrl+X`
- Press `Y` to confirm
- Press `Enter` to save

#### 6. Apply Configuration

**For Proxmox servers with running VMs:**
```sh
reboot
```
Changes will NOT take effect until server is rebooted. Do NOT use `systemctl restart networking` as this will disconnect all VM networks and require a reboot anyway to restore VM connectivity.

**For standalone Ubuntu servers (no VMs):**
```sh
systemctl restart networking
```

#### 7. Verify Configuration
After reboot, test connectivity:
```sh
ping -c 3 1.1.1.1
```

**For Proxmox**: Also verify VMs are reachable after reboot.

If ping works, your static IP is configured correctly.

---

### Network Configuration Fields Explained
- **address**: Your static IP with subnet mask (/24 = 255.255.255.0)
- **gateway**: Your router's IP address
- **dns-nameservers**: DNS servers for name resolution
- **dns-search**: Domain search suffix (if using Tailscale)
- **bridge-ports**: Physical interface connected to the bridge (Proxmox only)

### Notes
- **CRITICAL**: On Proxmox, always reboot instead of restarting networking service
- Restarting networking on Proxmox will disconnect all VM network interfaces
- Schedule maintenance window for Proxmox hosts
- Always verify network connectivity after changes
- **For Linux Mint/Ubuntu Desktop**: GUI method is recommended over command line
- For standalone Ubuntu servers without VMs, network restart is safe
