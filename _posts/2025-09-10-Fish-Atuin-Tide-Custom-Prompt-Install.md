---
layout: post
title: "Fish + Atuin + Tide + Custom Prompts for Linux terminals"
draft: false
date:  2025-09-10
description: This is an updated, cleaner and easier to follow outline on how to install Fish, Atuin, Tide and set custom left and right prompts in the terminal.  You still run Tide Configure to set up the Tide prompt initially.  
---
NOTE:  I set up all my machines to default to the Bash terminal and set up an alias f="fish" so that when I ssh in I can hit f and be in the Fish shell.  It seems to work better this way with scripting across many servers.  Also, Ctrl-D drops you out of Fish back to Bash and one more Ctrl-D exits the ssh session.

NOTE:  I have not fully tested this on CachyOS or any other Arch based system yet. In the process of doing CachyOS now and will update this article if needed.  

# Fish + Atuin + Tide Setup Commands
*Copy/paste these commands in sequence for each server*

## Step 1: Install Fish
```bash
# Ubuntu/Debian/Mint/Proxmox
sudo apt update && sudo apt install -y fish curl

# CachyOS  
sudo pacman -S fish curl
```

## Step 2: Add Bash Alias
```bash
echo 'alias f="fish"' >> ~/.bashrc
source ~/.bashrc
```

## Step 3: Install Atuin
```bash
bash <(curl --proto '=https' --tlsv1.2 -sSf https://setup.atuin.sh)
```

## Step 4: Switch to Fish and Install Everything
```bash
f
```

## Step 5: Install Fisher (Fish Package Manager)
```bash
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

## Step 6: Install Tide
```bash
fisher install IlanCosman/tide@v6
```

## Step 7: Install Atuin Fish Integration
```bash
fisher install atuinsh/atuin
```

## Step 8: Apply Your Clean Prompt Config
```bash
set -U tide_right_prompt_items context
set -U tide_left_prompt_items pwd git newline character
```

## Step 9: Test the New Prompt
```bash
exit
f
```

---

**Total time per server: ~90 seconds**

Your prompt will show:
```
╭─ /current/directory ─────────── user@servername ─╮
╰─❯ 
```