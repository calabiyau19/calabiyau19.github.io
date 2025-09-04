---
layout: post
title: "Add Tailscale to Ubuntu 24.04 VM"
date:  2025-03-08
---

### How to add Tailscale to a Ubuntu VM

Scenario: Ubuntu 24.04 server running in a VM on a proxmox host

#### Add Tailscale's GPG key

```sh
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
```

#### Add the tailscale repository

```sh
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```

#### Install Tailscale

```sh
sudo apt-get update && sudo apt-get install tailscale
```

#### Start Tailscale!

```sh
sudo tailscale up
```

#### Login to Tailscale account and accept connection
##### Done