---
layout: post
title: "Setup SSH access on Ubuntu server"
draft: false
date: 2025-10-09
description: A simple tutorial for setting up SSH access into a new Ubuntu server and adding SSH to your managment pc or laptop.  Bonus section on setting up SSH key pair acess as it is much easier longer term.  
---

## SSH Setup Guide for Ubuntu

A beginner-friendly guide to setting up SSH on an Ubuntu server VM and a management laptop for remote server administration.

### Overview

This guide will walk you through:

- Installing and configuring OpenSSH Server on your Ubuntu VM
- Setting up the SSH client on your management laptop
- Making your first SSH connection
- Optional: Setting up key-based authentication for enhanced security

### Prerequisites

- A fresh Ubuntu server VM with console access
- An Ubuntu laptop or desktop to use as your management machine
- Basic familiarity with the Linux command line

### Part 1: Setting Up SSH Server on Ubuntu VM

### Step 1: Log Into Your Server

Access your Ubuntu VM using the console or terminal interface provided by your virtualization platform.

### Step 2: Update Package List

```sh
sudo apt update
```

This ensures you're installing the latest version of available packages.

### Step 3: Install OpenSSH Server

```sh
sudo apt install openssh-server -y
```

The `-y` flag automatically answers "yes" to installation prompts.

### Step 4: Verify SSH Service is Running

```sh
sudo systemctl status ssh
```

You should see output showing `active (running)` in green text. Press `q` to exit the status view.

### Step 5: Enable SSH to Start on Boot

```sh
sudo systemctl enable ssh
```

This ensures SSH starts automatically whenever your server reboots.

### Step 6: Find Your Server's IP Address

```sh
ip a
```

Look for an IPv4 address in the output. It will typically be in one of these formats:

- `192.168.x.x` (common for local networks)
- `10.x.x.x` (common for private networks)
- Ignore `127.0.0.1` (that's localhost)

**Write down this IP address** - you'll need it to connect from your laptop.

### Step 7: Configure Firewall (Recommended)

```sh
sudo ufw allow ssh
sudo ufw enable
```

When prompted, type `y` to confirm. This allows SSH traffic through the firewall while blocking other unwanted connections.

## Part 2: Setting Up SSH Client on Management Laptop

### Step 1: Open Terminal

Launch the terminal application on your Ubuntu laptop.

### Step 2: Check for SSH Client

```sh
ssh -V
```

If you see version information (e.g., `OpenSSH_8.x`), the client is already installed. If not, install it:

```sh
sudo apt update
sudo apt install openssh-client -y
```

### Step 3: Make Your First Connection

```sh
ssh username@server-ip-address
```

**Replace:**

- `username` with your actual username on the server VM
- `server-ip-address` with the IP address you noted earlier

**Example:**

```sh
ssh john@192.168.1.100
```

### Step 4: Accept the Server Fingerprint

The first time you connect, you'll see a message like:

```
The authenticity of host '192.168.1.100' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter.

### Step 5: Enter Your Password

When prompted, enter the password for your user account on the server. Note that you won't see any characters as you type (this is normal for password entry in Linux).

### Step 6: You're Connected

You should now see your server's command prompt. You're successfully connected via SSH!

### Disconnecting

To end your SSH session and return to your laptop's terminal:

```sh
exit
```

## Part 3: Key-Based Authentication (Optional but Recommended)

Key-based authentication is more secure than passwords and more convenient for frequent connections.

### Step 1: Generate SSH Key Pair (On Your Laptop)

```sh
ssh-keygen -t ed25519
```

**Prompts you'll see:**

- **Save location:** Press Enter to accept the default (`~/.ssh/id_ed25519`)
- **Passphrase:** Either enter a passphrase for additional security, or press Enter twice for no passphrase

### Step 2: Copy Your Public Key to the Server

```sh
ssh-copy-id username@server-ip-address
```

Enter your server password when prompted. This will be the last time you need to enter it.

### Step 3: Test Key-Based Login

```sh
ssh username@server-ip-address
```

You should now connect without being asked for your password (or only for your key passphrase if you set one).

## Troubleshooting

### Can't Connect to Server

- Verify the server IP address is correct
- Ensure SSH service is running on the server: `sudo systemctl status ssh`
- Check firewall settings: `sudo ufw status`
- Verify you're using the correct username

### Connection Refused

- SSH service may not be running: `sudo systemctl start ssh`
- Firewall may be blocking port 22: `sudo ufw allow ssh`

### Permission Denied

- Double-check your username and password
- Ensure your user account exists on the server

## Next Steps

Now that you have SSH set up, you can:

- Create an SSH config file (`~/.ssh/config`) for connection shortcuts
- Learn about SSH port forwarding for accessing remote services
- Explore SCP and RSYNC for file transfers
- Set up SSH key management for multiple servers

## Security Best Practices

- Use key-based authentication instead of passwords when possible
- Keep your SSH client and server software updated
- Consider changing the default SSH port (22) to reduce automated attacks
- Disable root login via SSH
- Use strong passphrases for SSH keys
- Regularly audit who has access to your servers

---

**Document Version:** 1.0  
**Last Updated:** October 2025
