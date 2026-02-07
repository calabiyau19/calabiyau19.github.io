---
layout: post
draft: false
title: "Cloning Ubuntu 24.04 VMs on Proxmox Without Conflicts"
date: 2026-01-25
description: "This post documents the step by step process to create a new Ubuntu 24.04 VM on my Proxmox hypervisor from an existing Ubuntu 24.04 VM that is fully up to date and contains extras like Atuin, Fish, frequently run scripts, Docker, etc. This is a little tricky because the existing VM is NOT a Proxmox template therefor once a copy is created (on the same Proxmox server) there are several imporant things that must be done to make it a fully usable VM ready for a new application to be installed into."
---

## Overview

This guide documents the process of cloning a fully configured Ubuntu 24.04 "golden image" VM on the same Proxmox hypervisor while ensuring the clone operates independently without IP address conflicts or other network issues.

**Important:** This guide covers cloning a regular, fully-configured VM (not a Proxmox template). The base VM remains a working VM that you can continue to update and maintain, then clone again in the future when you need new VMs with your standard configuration.

## Prerequisites

- A working Ubuntu 24.04 VM on Proxmox configured with your standard tools (SSH, Docker, Atuin, etc.)
- This base/golden image VM should be shut down temporarily for cloning
- Access to Proxmox web interface or command line
- Understanding of your network IP addressing scheme

## Step 1: Prepare the Golden Image VM for Cloning

Before cloning, shut down your base VM to ensure a consistent copy.

### 1.1 Shut Down the Base VM

From Proxmox host CLI (replace 100 with your VM ID):
```bash
qm shutdown 100
```

Or from within the VM:
```bash
sudo shutdown -h now
```

**Verify:** In Proxmox web interface, confirm VM status shows "stopped"

### 1.2 Optional: Update the Golden Image

Before cloning, you may want to ensure your golden image has the latest updates.

SSH into the base VM before shutting it down:
```bash
sudo apt update
```
```bash
sudo apt upgrade -y
```
```bash
sudo apt autoremove -y
```

Then shut down:
```bash
sudo shutdown -h now
```

**Note:** You can skip cleaning operations like clearing machine-id or bash history since this is a working VM you'll continue to use and update.

## Step 2: Clone the VM in Proxmox

### 2.1 Using Proxmox Web Interface

1. Navigate to your Proxmox datacenter view
2. Select the base/golden image VM (e.g., VM 100)
3. Right-click and select **"Clone"** or click the "More" button and select "Clone"
4. Configure clone settings:
   - **VM ID:** Choose a new unique ID (e.g., 101)
   - **Name:** Give it a descriptive name (e.g., `ubuntu-docker-host`)
   - **Mode:** Select **"Full Clone"** (not linked clone)
   - **Target Storage:** Select appropriate storage (usually same as source)
5. Click **"Clone"** to begin the process

**Verify:** Wait for cloning to complete (this may take several minutes), then confirm the new VM appears in your VM list.

### 2.2 Using Proxmox Command Line

Clone VM 100 to new VM 101 with full clone:
```bash
qm clone 100 101 --name ubuntu-docker-host --full
```

Check clone status:
```bash
qm status 101
```

**Verify:** Run `qm list` to see both VMs in the list (both should show "stopped").

## Step 3: Start VMs in Correct Order

**CRITICAL:** After cloning, you must start the VMs in the correct order to prevent IP address conflicts.  

**NOTE:** Any VMs that are shutdown should be started before bringing the new VM online or else these VMs might lose their existing IP address if they are not set with static addresses and use your DHCP server instead. Just saves time later tidying things up if IP addresses do "jump". 

### 3.1 Start the Original/Golden Image VM First

This is essential to ensure your original VM reclaims its IP address (especially important with DHCP).

From Proxmox host:
```bash
qm start 100
```

Or use web interface: select VM 100, click "Start"

**Verify:** 

Wait 30-60 seconds for boot, then check it's running:
```bash
qm status 100
```

Should show: status: running

**Optional verification:** SSH into the original VM to confirm it has its expected IP:
```bash
ssh user@192.168.1.100
```
```bash
ip addr show
```

### 3.2 Now Start the Cloned VM

Only after the original VM is fully booted and has claimed its IP address, start the clone.

From Proxmox host:
```bash
qm start 101
```

Or use web interface: select VM 101, click "Start"

**Verify:**
```bash
qm status 101
```

Should show: status: running

## Step 4: Configure the Cloned VM

Now that both VMs are running, you need to access the clone via console to discover its IP address and reconfigure it.

### 4.1 Access Clone via Proxmox Console

1. In Proxmox web interface, select the cloned VM (101)
2. Click **"Console"** to open no VNC console
3. Log in with the same credentials as your original VM

**Note:** You cannot SSH yet because you don't know the clone's IP address.

### 4.2 Discover the Clone's IP Address

Check the IP address assigned to the clone:
```bash
ip addr show
```

Look for the inet line under your network interface (usually ens18). Example output:
```
inet 192.168.1.152/24 brd 192.168.1.255 scope global dynamic ens18
```

Note this IP address - you'll use it for SSH in the next steps.

**Verify:** The clone should have a different IP from your original VM (192.168.1.100 in our example).

### 4.3 Change Hostname

Check current hostname (will be same as original VM):
```bash
hostnamectl
```

Set new hostname:
```bash
sudo hostnamectl set-hostname ubuntu-docker-host
```

Edit /etc/hosts to match:
```bash
sudo nano /etc/hosts
```

Update `/etc/hosts`:
```
127.0.0.1       localhost
127.0.1.1       ubuntu-docker-host

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

**Verify:**
```bash
hostnamectl
```

Should show new hostname.
```bash
hostname
```

Should also show new hostname.

### 4.4 Optional: Configure Static IP

If you want to assign a specific static IP to the clone (instead of using DHCP), update netplan.

Check current network configuration:
```bash
cat /etc/netplan/00-installer-config.yaml
```

Edit netplan configuration:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example netplan for static IP:
```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 192.168.1.105/24  # Change to desired static IP
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
          - 8.8.8.8
```

Apply netplan changes:
```bash
sudo netplan apply
```

Verify new IP:
```bash
ip addr show
```

**Verify:** Test connectivity:
```bash
ping -c 4 192.168.1.1
```
```bash
ping -c 4 8.8.8.8
```
```bash
nslookup google.com
```

**Note:** If you assign a static IP, you may lose console connection briefly. Reconnect via console if needed.

### 4.5 Regenerate Machine-ID (Important!)

The cloned VM will have the same machine-id as the original, which can cause issues with systemd and other services.

Check current machine-id:
```bash
cat /etc/machine-id
```

Remove old machine-id files:
```bash
sudo rm /etc/machine-id
```
```bash
sudo rm /var/lib/dbus/machine-id
```

Regenerate new unique machine-id:
```bash
sudo systemd-machine-id-setup
```

Create dbus symlink:
```bash
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
```

Verify new machine-id was created:
```bash
cat /etc/machine-id
```

**Verify:** The machine-id should be a new 32-character hex string, different from the original VM.

**Note:** Some services may need to be restarted after changing machine-id. A reboot is recommended after all configuration changes.

### 4.6 Clear Cached SSH Fingerprint on Management Laptop

When you SSH to the cloned VM from your management laptop, (sometimes) you'll get a warning because the IP address now has different SSH host keys than before.

From your management laptop (lpt-hp), remove the old cached fingerprint (replace with the clone's IP):
```bash
ssh-keygen -R 192.168.1.152   #the new VM IP address
```

**Verify:** Now SSH into the clone normally:
```bash
ssh user@192.168.1.152   #the new VM IP address
```

You'll be prompted to accept the new fingerprint. Type `yes` to accept.

### 4.7 Reboot to Apply All Changes

After making all configuration changes, reboot the cloned VM:
```bash
sudo reboot
```

**Verify:** After reboot, SSH into the clone using its IP address.

From your management laptop:
```bash
ssh user@192.168.1.152
```

Verify hostname:
```bash
hostname
```

Verify machine-id is unique:
```bash
cat /etc/machine-id
```

## Step 5: Final Verification Steps

### 5.1 SSH Access from Management Laptop

SSH to the cloned VM using its IP:
```bash
ssh user@192.168.1.152
```

Verify hostname is correct:
```bash
hostname
```

Should show: ubuntu-docker-host

Verify machine-id is unique:
```bash
cat /etc/machine-id
```

Should be different from original VM.

Check SSH fingerprint:
```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Should be different from original VM.

### 5.2 Network Connectivity Tests

Test DNS resolution:
```bash
nslookup google.com
```

Test outbound connectivity:
```bash
curl -I https://www.google.com
```

Check listening services:
```bash
sudo ss -tlnp | grep :22
```

### 5.3 Verify No IP Conflicts

From your management laptop, verify both VMs are accessible.

Ping original/golden image VM:
```bash
ping -c 4 192.168.1.100
```

Ping cloned VM:
```bash
ping -c 4 192.168.1.152
```

SSH to original VM:
```bash
ssh user@192.168.1.100
```
```bash
hostname
```

Should show original hostname.
```bash
exit
```

SSH to cloned VM:
```bash
ssh user@192.168.1.152
```
```bash
hostname
```

Should show new hostname.
```bash
exit
```

**Verify:** Both VMs should respond without conflicts or timeouts.

### 5.4 Verify Installed Services

Check Docker (from your golden image):
```bash
sudo systemctl status docker
```
```bash
sudo docker ps
```

Check SSH:
```bash
sudo systemctl status ssh
```

Check Atuin (if installed):
```bash
systemctl --user status atuin
```

List all enabled services:
```bash
systemctl list-unit-files --state=enabled
```

### 5.5 Update LibreNMS or Monitoring (if applicable)

If you use LibreNMS or other monitoring:

1. Add the new cloned VM as a device with its new hostname and IP
2. Verify SNMP or other monitoring agents are working
3. Update any documentation or inventory systems

## Step 6: Golden Image Maintenance

### 6.1 Keeping Your Golden Image Updated

Your golden image VM (ID 100) should be periodically updated.

SSH into the golden image VM:
```bash
ssh user@192.168.1.100
```

Update packages:
```bash
sudo apt update
```
```bash
sudo apt upgrade -y
```
```bash
sudo apt autoremove -y
```

Update Docker images if you have base images:
```bash
sudo docker image prune -a
```

Restart if kernel was updated:
```bash
sudo reboot
```

**Recommendation:** Update your golden image monthly or before creating new clones.

### 6.2 Document Your VM Inventory

Keep track of your VMs and their purposes:

| VM ID | Hostname | IP Address | Purpose | Cloned From | Created |
|-------|----------|------------|---------|-------------|---------|
| 100 | ubuntu-base | 192.168.1.100 | Golden Image | Original | 2026-01-15 |
| 101 | ubuntu-docker-host | 192.168.1.152 | Docker Host | VM 100 | 2026-01-25 |
| 102 | ubuntu-test | 192.168.1.153 | Testing | VM 100 | 2026-01-25 |

### 6.3 When to Create a New Golden Image

Consider creating a fresh golden image when:
- Ubuntu releases a new LTS version
- You've made so many changes to the current golden image that it's becoming unwieldy
- You want to standardize on a different set of base tools
- The current golden image has accumulated technical debt

## Best Practices

1. **Always shut down the golden image VM before cloning** to ensure a consistent snapshot
2. **Always start the original VM first** after cloning to ensure it reclaims its IP address
3. **Always use full clones** for production VMs (not linked clones)
4. **Use Proxmox console** to discover the clone's IP address before attempting SSH
5. **Change hostname immediately** after first boot of the clone
6. **Regenerate machine-id and SSH keys** to ensure each VM is unique
7. **Document your IP addressing scheme** to avoid confusion
8. **Keep a VM inventory spreadsheet/document** tracking all clones and their purposes
9. **Update your golden image regularly** but test updates before cloning
10. **Use descriptive VM names** that indicate purpose (e.g., ubuntu-docker-host, ubuntu-monitoring)
11. **Consider DHCP reservations** in your router for cloned VMs you want to keep long-term
12. **Test SSH connectivity** from your management laptop before considering the clone complete
13. **Update monitoring systems** (LibreNMS, etc.) when adding new VMs
14. **Take snapshots** of important cloned VMs after initial configuration

## Troubleshooting

### Wrong VM Started First

If you accidentally started the clone before the original:

1. Shut down BOTH VMs immediately:
```bash
qm shutdown 100
```
```bash
qm shutdown 101
```

2. Wait for both to fully stop

3. Start the original (100) first:
```bash
qm start 100
```

4. Wait 60 seconds for it to fully boot and claim its IP

5. Then start the clone (101):
```bash
qm start 101
```

### IP Address Conflict Detected

If you see ARP conflicts or both VMs have the same IP:

1. Check which VM has which IP.

Access each via Proxmox console.

VM 100 console:
```bash
ip addr show
```

VM 101 console:
```bash
ip addr show
```

2. If the clone has the original's IP, restart both VMs in correct order (see above)
3. If using static IPs, immediately configure the clone with a different IP via console

### Clone Won't Start

Check VM configuration:
```bash
qm config 101
```

Check Proxmox logs:
```bash
tail -f /var/log/syslog | grep kvm
```

Verify disk is accessible:
```bash
qm list
```

### Can't SSH After Cloning

- Verify IP address with `ip addr show` in console
- Check SSH service: `sudo systemctl status ssh`
- Verify firewall: `sudo ufw status`
- Check SSH key fingerprint changed

### Machine-ID Issues

If services complain about duplicate machine-id:
```bash
sudo rm /etc/machine-id
```
```bash
sudo rm /var/lib/dbus/machine-id
```
```bash
sudo systemd-machine-id-setup
```
```bash
sudo systemctl reboot
```

## Advanced: Scripting the Post-Clone Configuration

For frequent cloning, you can create a script to automate the post-clone configuration:
```bash
#!/bin/bash
# post-clone-config.sh
# Run this script on a newly cloned VM

NEW_HOSTNAME="$1"

if [ -z "$NEW_HOSTNAME" ]; then
    echo "Usage: $0 <new-hostname>"
    exit 1
fi

echo "Configuring cloned VM with hostname: $NEW_HOSTNAME"

# Change hostname
sudo hostnamectl set-hostname "$NEW_HOSTNAME"

# Update /etc/hosts
sudo sed -i "s/127.0.1.1.*/127.0.1.1\t$NEW_HOSTNAME/g" /etc/hosts

# Regenerate machine-id
sudo rm -f /etc/machine-id /var/lib/dbus/machine-id
sudo systemd-machine-id-setup
sudo ln -sf /etc/machine-id /var/lib/dbus/machine-id

# Regenerate SSH host keys
sudo rm -f /etc/ssh/ssh_host_*
sudo ssh-keygen -A
sudo systemctl restart ssh

echo "Configuration complete. Current IP address:"
ip addr show | grep "inet " | grep -v "127.0.0.1"

echo ""
echo "Please reboot the VM to ensure all changes take effect:"
echo "  sudo reboot"
```

Save this script to your golden image VM at `/usr/local/bin/post-clone-config.sh` and make it executable:
```bash
sudo chmod +x /usr/local/bin/post-clone-config.sh
```

After cloning and starting the VMs in correct order, run this script via console:
```bash
sudo /usr/local/bin/post-clone-config.sh ubuntu-new-server
```

## Related Documentation

- [Ubuntu Server Guide: Net plan](https://netplan.io/)
- [Proxmox VE: QEMU/KVM Virtual Machines](https://pve.proxmox.com/wiki/Qemu/KVM_Virtual_Machines)
- [Ubuntu: Network Configuration](https://ubuntu.com/server/docs/network-configuration)

## Conclusion

Cloning a fully-configured Ubuntu VM on Proxmox is an efficient way to deploy new servers with your standard tools and configurations. The process is straightforward but requires careful attention to the startup sequence and post-clone configuration to avoid IP conflicts and ensure each VM has a unique identity.

### Critical Steps Summary

1. **Shut down** the golden image VM before cloning
2. **Clone** using Proxmox (full clone, not linked)
3. **Start the original VM first** - this is critical for IP address management
4. **Then start the clone** - it will get a new IP via DHCP
5. **Use console** to discover the clone's IP address
6. **Reconfigure** the clone: hostname, machine-id, SSH keys
7. **Verify** both VMs are accessible and functioning independently

### Key Principles

- The original/golden image VM always starts first after cloning
- Always use full clones for independent VMs
- Use console access when you don't know the IP yet
- Change hostname and regenerate unique identifiers immediately
- Verify connectivity before considering the clone production-ready
- Keep your golden image updated and well-documented

By following this methodical process and verifying at each stage, you can reliably deploy new VMs from your golden image while maintaining network stability and avoiding conflicts.

---

*Last updated: 2026-01-25*