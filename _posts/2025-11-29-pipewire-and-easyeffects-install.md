---
layout: post
title: "Pipewire and EasyEffects Laptop Sound Utility Installation"  
draft: false
date: 2025-11-29
description: "Updated my Linux Mint 22.2 laptop to use pipewire instead of pulse audio that I've been using for almost a year due to original sound issues with the laptop. This document also details the installation and setup of a program called Easy Effects which is a sound program add-on to be able to fine-tune the sound coming from your speakers in my case laptop speakers."
---

## Overview

This guide covers installing PipeWire audio server and Easy Effects on Linux Mint 22.2, replacing PulseAudio with the modern PipeWire stack.

## Prerequisites

- Linux Mint 22.2 (or similar Ubuntu-based distribution)
- Timeshift backup created before proceeding
- Terminal access with sudo privileges

## Step 1: Check Current Audio Server

First, verify which audio server is currently running:
```sh
inxi -A
```

If you see `PulseAudio` listed, you'll need to switch to PipeWire. If you already see `PipeWire`, skip to Step 4.

## Step 2: Update System

Ensure your system is fully updated:
```sh
sudo apt update && sudo apt upgrade -y
```

## Step 3: Install PipeWire

Install the complete PipeWire stack:
```sh
sudo apt install pipewire pipewire-pulse wireplumber pipewire-alsa
```

This will:
- Remove PulseAudio packages
- Install PipeWire core components
- Install PulseAudio compatibility layer (pipewire-pulse)
- Install wire plumber (session manager)
- Install ALSA compatibility

## Step 4: Enable PipeWire Services

Enable and start the PipeWire services:
```sh
systemctl --user --now enable pipewire pipewire-pulse wireplumber
```

Verify PipeWire is running:
```sh
inxi -A
```

You should now see `Server-1: PipeWire v: 1.0.5 status: active`

## Step 5: Install Bluetooth Audio Support

Install the PipeWire Bluetooth module:
```sh
sudo apt install libspa-0.2-bluetooth
```

Restart PipeWire services to load the Bluetooth module:
```sh
systemctl --user restart pipewire pipewire-pulse wireplumber
```

## Step 6: Fix Flatpak Application Audio Access

If you use Flatpak applications (like Brave browser), restart the desktop portal service:
```sh
systemctl --user restart xdg-desktop-portal
```

**Important:** After restarting the portal, completely close and reopen any Flatpak applications for them to detect the audio devices.

## Step 7: Test Audio

Test all audio devices:
- Laptop speakers
- Microphone
- Bluetooth devices (speakers, earbuds, hearing aids)

Verify microphone detection:
```sh
pactl list sources short
```

You should see your audio input devices listed.

## Step 8: Install Easy Effects

Install Easy Effects audio processing application:
```sh
sudo apt install easyeffects
```

Launch Easy Effects:
```sh
easyeffects
```

## Step 9: Install Community Presets

Download community presets from:
- Official wiki: https://github.com/wwmm/easyeffects/wiki/Community-presets
- Popular collection: https://github.com/JackHack96/EasyEffects-Presets

**To install presets:**

1. Download the preset repository (click Code → Download ZIP)
2. Extract the `.json` preset files
3. In Easy Effects, click hamburger menu (top right)
4. Select "Import Preset"
5. Choose the `.json` files
6. Presets will appear in the left sidebar

**Recommended presets for laptop speakers:**
- Bass Enhancing + Perfect EQ
- Loudness Equalizer - I did not end sup using this one due to distortion

## Troubleshooting

### Flatpak Applications Can't Detect Microphone

If Flatpak applications (Brave, etc.) can't detect the microphone after installing PipeWire:

1. Restart the desktop portal:
```sh
systemctl --user restart xdg-desktop-portal
```

2. **Completely close all instances** of the application (check all virtual desktops - IMPORTANT)

3. Restart the application fresh

### Bluetooth Devices Connect But No Audio

Ensure the Bluetooth module is installed:
```sh
sudo apt install libspa-0.2-bluetooth
systemctl --user restart pipewire pipewire-pulse wireplumber
```

Re-pair Bluetooth devices after restart.

### Verify PipeWire Is Running

Check service status:
```sh
systemctl --user status pipewire pipewire-pulse wireplumber
```

All three services should show `active (running)`.

## Using Easy Effects

Easy Effects has three main tabs:

- **Output**: Apply effects to speaker audio (equalizer, bass boost, etc.)
- **Input**: Apply effects to microphone audio (noise reduction, etc.)
- **PipeWire**: View and manage audio routing

To add effects:
1. Go to Output or Input tab
2. Scroll to bottom of window
3. Click "Effects" section
4. Click "+" to add effects from ~20 available options

Activate presets by clicking them in the left sidebar.

## Notes

- PipeWire replaced PulseAudio starting with Linux Mint 22
- The `pipewire-pulse` package provides backward compatibility for PulseAudio applications
- Easy Effects requires PipeWire and will not work with PulseAudio
- System reboots are generally not required after installation, but service restarts are necessary

## References

- Easy Effects GitHub: https://github.com/wwmm/easyeffects
- Community Presets: https://github.com/JackHack96/EasyEffects-Presets
- PipeWire Documentation: https://docs.pipewire.org/
EOF