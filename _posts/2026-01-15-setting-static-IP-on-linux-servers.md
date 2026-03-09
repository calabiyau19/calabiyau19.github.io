---
layout: post
draft: false
title: "How to set a static IP address on a linux server or desktop"
date: 2026-01-15
description: "This tutorial shows how to set a static IP address on Ubuntu 24.04 servers using netplan, and on Linux Mint desktops using the NetworkManager GUI or command line."
---

## Setting a Static IP on Ubuntu 24.04 Servers and Linux Mint Desktops

### Overview

This guide covers setting a static IP address on Ubuntu 24.04 servers (which use **netplan**) and on Linux Mint or Ubuntu desktop systems (which use **NetworkManager**). The `/etc/network/interfaces` method used in older Ubuntu releases does not apply to Ubuntu 24.04.

---

## For Ubuntu 24.04 Servers (Netplan)

This applies to standalone Ubuntu VMs and servers. Proxmox hosts use a different method and are not covered here.

### Prerequisites

- `sudo` access on the server
- The static IP address you want to assign
- SSH access or Proxmox console access

---

### Steps

#### 1. Find Your Default Gateway

Before editing anything, confirm your current gateway address:

```sh
ip route show
```

Look for the line beginning with `default via` — the IP address that follows is your gateway. Note it down, you will need it shortly.

#### 2. Find Your Network Interface Name

```sh
ip link show
```

Look for your active interface — commonly `ens18` on Proxmox VMs. Note the name.

#### 3. View the Current Netplan Configuration

```sh
sudo cat /etc/netplan/50-cloud-init.yaml
```

You should see something like:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true
```

This confirms the interface name and that DHCP is currently active.

#### 4. Edit the Netplan Configuration

```sh
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the entire contents with the following, substituting your values:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 192.168.1.XXX/24
      routes:
        - to: default
          via: 192.168.1.YYY
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Replace:
- `ens18` with your interface name from Step 2
- `192.168.1.XXX` with your desired static IP
- `192.168.1.YYY` with your gateway address from Step 1

Save and exit: `Ctrl+X`, then `Y`, then `Enter`.

#### 5. Apply the Configuration

```sh
sudo netplan apply
```

Your SSH session will drop immediately when this runs — that is expected behavior, not an error. The IP address has changed.

#### 6. Reconnect at the New IP

Open a new terminal and SSH to the new static IP:

```sh
ssh mark@192.168.1.XXX
```

#### 7. Verify the New IP is Active

```sh
ip addr show
```

Confirm your new static IP appears next to your interface name with `scope global`.

#### 8. Verify SSH Service is Running

In some cases, particularly on template VMs, the SSH service may be inactive after the IP change. Check its status:

```sh
sudo systemctl status ssh
```

If it shows inactive or disabled, enable and start it:

```sh
sudo systemctl enable ssh --now
```

Then reboot the VM to confirm SSH starts automatically on future boots:

```sh
sudo reboot
```

Reconnect after reboot and verify SSH is working at the new IP.

#### 9. Test Connectivity

```sh
ping -c 3 1.1.1.1
```

If ping succeeds, your static IP is configured correctly.

---

## For Linux Mint / Ubuntu Desktop (NetworkManager)

### GUI Method (Recommended)

1. Click the network icon in your system tray
2. Click **Network Settings** or **Edit Connections**
3. Find your active connection and click the gear icon or **Edit**
4. Go to the **IPv4 Settings** tab
5. Change method from **Automatic (DHCP)** to **Manual**
6. Click **Add** and enter:
   - **Address**: your desired static IP
   - **Netmask**: `255.255.255.0` (or `24`)
   - **Gateway**: your gateway address (run `ip route show` to confirm)
7. In the **DNS servers** field enter: `1.1.1.1, 8.8.8.8`
8. Click **Save**
9. Disconnect and reconnect to the network

### Command Line Method (Alternative)

First, find your connection UUID:

```sh
nmcli connection show
```

Then apply the static IP settings using your UUID:

```sh
nmcli connection modify <UUID> ipv4.method manual ipv4.addresses 192.168.1.XXX/24 ipv4.gateway 192.168.1.YYY ipv4.dns "1.1.1.1 8.8.8.8"
```

Bring the connection down and back up to apply:

```sh
nmcli connection down <UUID> && nmcli connection up <UUID>
```

Verify:

```sh
ip addr show | grep "inet "
```

**Note**: If you see warnings about duplicate connection names, use the UUID instead of the connection name in all commands.

---

### Configuration Fields Explained

- **dhcp4: false** — Disables DHCP, so the static address is used instead
- **addresses** — Your static IP with subnet mask (`/24` = `255.255.255.0`)
- **routes / via** — Your router's gateway IP address
- **nameservers** — DNS servers used for name resolution
