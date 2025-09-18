---
title: Setting up the Terminal Post Server Installation in Linux
date: 2025-09-18
description: After every installation of an Ubuntu server in a VM that I will be using to hold a Docker style application, this is how I set up the terminal so that every one of my servers operates with the same terminal configuration.  It got a little crazy with twenty different servers and pcs trying to remember which ones had only sh or which ones had the full stack of Fish, Atuin, Tide and custom prompts. So I made them all the same. 
---

# Setting up the Terminal Post Server Installation in Linux

## Overview

This template provides a standardized terminal configuration for Ubuntu servers running Docker applications in VMs. The setup maintains consistency across all server instances while providing enhanced shell functionality. The idea is that I will ssh (with key pairs) into the linux based machines and land in a Bash shell by default - which helps with running scripts across the network.  Then by simple pressing "f" I am switched into Fish.  In either shell, the history via the up arrow is maintained with Atuin.  

## Shell Configuration Template (Post-Update Ubuntu Server)

This template accomplishes the following:

* Keeps **Bash as the default shell**
* Installs **Fish**
* Installs **Atuin** with sync + shared history
* Installs **Fisher + Tide prompt**
* Sets custom **Tide left/right prompts**
* Ensures **Atuin is integrated with both Bash and Fish**
* Adds `f` alias to start Fish from Bash

## Step-by-step Shell Setup Template

Run the following commands **as your regular user** (not root).

### 1. Install Shell Tools

```sh
sudo apt update && sudo apt install -y fish curl git
```

### 2. Set Bash Alias for Launching Fish

```sh
echo "alias f='fish'" >> ~/.shrc
source ~/.shrc
```

### 3. Enter the Fish Shell

```sh
f
```

### 4. Install Fisher & Tide (inside Fish)

```sh
curl -sL https://git.io/fisher | source && fisher install jorgebucaran/fisher
fisher install IlanCosman/tide
```

### 5. Set Tide Prompt Configuration

```sh
set -U tide_left_prompt_items pwd git newline character
set -U tide_right_prompt_items context
```

### 6. Install Atuin (inside Fish)

```sh
curl -s https://raw.githubusercontent.com/atuinsh/atuin/main/install.sh | sh
```

### 7. Configure Atuin Sync & Import History

```sh
atuin import auto
atuin register   # or `atuin login` if you already have an account
```

### 8. Enable Atuin for Fish

```sh
atuin init fish | source
mkdir -p ~/.config/fish/conf.d
atuin init fish > ~/.config/fish/conf.d/atuin.fish
```

### 9. Exit Fish Shell

```sh
exit
```

### 10. Enable Atuin for Bash

```sh
echo 'eval "$(atuin init bash)"' >> ~/.bashrc
source ~/.bashrc
```

## Result

Upon completion, you will have:

* **Bash as your default shell**
* The ability to enter **Fish** with `f`
* Consistent **Tide** prompt
* Full **Atuin history** across Bash & Fish

This configuration ensures all your Ubuntu server VMs have identical terminal environments, making administration and development workflows consistent across your infrastructure.