---
layout: post
title: "How to copy a empty but modifed 'template' VM from Nu5 to Nuc3"
draft: false
date: 2025-11-30
description: "I maintain two Proxmox servers and keep a fully updated Ubuntu Server 24.04 template VM ready for deployment. This template includes SSH with my key pair, Fish shell, ATUIN, Docker, Docker Compose, and my user configured in the Docker group. When deploying a new VM from this template, I only need to update the IP address, configure a static DHCP reservation, change the hostname, and install the target application. I keep the master template on NUC5. For deployments on NUC5, I clone the template directly. For deployments on NUC3, I copy the template over first, then complete the configuration. This document outlines the steps to transfer the template between servers—a process I perform infrequently enough to warrant documented procedures."
---


## Complete Guide: Copy and Configure a VM from One Proxmox Host to Another

This guide covers the complete process of copying a VM from one Proxmox host to another using vzdump backup and restore, then configuring the new VM with proper hostname, IP address, and management access.

## Overview

This method uses vzdump backup, SCP file transfer, and qmrestore. Works on any Proxmox host and requires NO clustering. After the VM is restored, several cleanup steps are needed to avoid conflicts and properly configure the new VM for its new role.

## Part 1: Copying the VM

### Step 1: On Source Host (NUC5) - Create Backup of VM 890

```sh
vzdump 890 --mode stop --compress zstd --storage nvme1tb-backup
```

This produces:

```sh
/mnt/nvme1tb/backup/dump/vzdump-qemu-890-YYYY_MM_DD-HH_MM_SS.vma.zst
```

### Step 2: Copy Backup from Source to Destination Host

```sh
scp /mnt/nvme1tb/backup/dump/vzdump-qemu-890-*.vma.zst root@192.168.1.121:/root/
```

### Step 3: (Optional) Decompress the Backup on Destination Host (NUC3)

Proxmox can restore compressed or decompressed, but if you want to decompress:

```sh
zstd -d vzdump-qemu-890-*.vma.zst
```

You will now have:

```sh
vzdump-qemu-890-xxxx.vma
```

### Step 4: On Destination Host (NUC3) - Restore the VM into local-lvm

(Your storage on NUC3 for VM disks)

If restoring the compressed file:

```sh
qmrestore vzdump-qemu-890-*.vma.zst 890 --storage local-lvm
```

If restoring the decompressed .vma:

```sh
qmrestore vzdump-qemu-890-*.vma 890 --storage local-lvm
```

The VM is now restored but DO NOT start it yet. Proceed to Part 2 for cleanup steps.

## Part 2: Post-Clone VM Cleanup

After cloning a VM using vzdump/restore, the new VM is an exact copy, which creates configuration issues that must be resolved.

### Problems After Cloning

1. **Hostname** - The new VM has the same hostname as the original
2. **IP Address Conflict** - Both VMs have the same MAC address, so they request the same DHCP IP lease

### Step 5: Change MAC Address on New VM (Prevents IP Conflict)

On the destination Proxmox host (NUC3), with the VM still shut down:

```sh
qm set 890 -net0 virtio,bridge=vmbr0
```

This auto-generates a new MAC address for the VM.

Alternatively, use the Proxmox web UI:
- Select the VM
- Hardware → Network Device → Edit
- Remove or regenerate the MAC address
- Click OK

### Step 6: Start VMs in Correct Order

1. Start the original VM on the source host (NUC5) - it reclaims its existing IP
2. Start the new VM on the destination host (NUC3) - it gets a fresh DHCP IP

```sh
qm start 890
```

### Step 7: Set DHCP Reservation for New VM

Check what IP the new VM received:

```sh
ip addr show
```

Log into your router/modem admin interface and create a DHCP reservation for the new VM's MAC address with its new IP.

### Step 8: Change Hostname on New VM

SSH into the new VM and set the new hostname:

```sh
sudo hostnamectl set-hostname ghostfolio-vm
```

Update /etc/hosts to match:

```sh
sudo nano /etc/hosts
```

Change the line with the old hostname to:

```
127.0.1.1    ghostfolio-vm
```

Verify the change:

```sh
hostnamectl
```

Log out and back in for the shell prompt to update.

### Step 9: Copy Scripts from Management Host

From the management host (lpt-hp), copy your scripts directory to the new VM:

```sh
scp -r ~/Scripts mark@192.168.1.155:~/
```

Or from any machine:

```sh
scp -r mark@192.168.1.100:~/Scripts mark@192.168.1.155:~/
```

### Step 10: Configure SSH Access from Management Host

From the management host (lpt-hp), add its public key to the new VM:

```sh
ssh-copy-id mark@192.168.1.155
```

This prompts for the password once, then sets up passwordless SSH access.

Test the connection:

```sh
ssh mark@192.168.1.155
```

The last step for a permanent addition to the network is to update the spreadsheet, add it to Tabby and add it to the tailnet.  So the Tailscale part first.

## Complete

The VM originally on NUC5 (192.168.1.125) is now fully restored and running on NUC3 (192.168.1.121) with proper configuration:

- Original VM on NUC5: Retains original IP address and hostname
- New VM on NUC3: New IP (192.168.1.155), new MAC address, new hostname (ghostfolio-vm)
- Management access from lpt-hp configured and working

## Key Takeaway

Virtual machine MAC addresses are configuration settings in the hypervisor, not tied to physical hardware. This makes them easy to change to prevent conflicts when cloning VMs - a significant advantage of virtualization.
