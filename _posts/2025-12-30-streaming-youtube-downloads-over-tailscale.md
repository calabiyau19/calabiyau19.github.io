---
layout: post
draft: false
title: "How to stream downloaded YouTube videos over Tailscale, but run minimized"
date: 2025-12-30
description: The project goal was to be able to stream YouTube videos that were downloaded using MeTube, over a tailscale connection where the player streaming the video was minimized, so just the audio could be listened to instead of having to watch the video as well. The videos were downloaded to an external SSD attached to a proxmox server, and the whole thing was turned into a network Samba server for file storage.  The videos were the first item put on it.
---
TLDR: While we did try a couple different solutions, FE file explorer pro, Copyparty, direct share access, in the end, and I decided to stay with Infuse, which is an IOS app, since it will play videos over tailscale minimized. It just takes a while to load. FE File Explorer Pro works wonderfully, but you cannot minimize the screen or turn the screen off as the video will stop - which is the behavior we were trying to stop. Still trying to determine if there is a way around this.

# Streaming YouTube Videos Over Tailscale with Background Audio

The goal was to stream YouTube videos downloaded with MeTube, stored on samba-storage-vm (Ubuntu VM on Proxmox), accessible remotely over Tailscale, with the ability to play audio in the background while the phone screen is off or while using other apps.

## Original Problem

The Infuse app over Tailscale had severe delays: 45 seconds to browse folders, 45 seconds to list videos, and 60 seconds to start playback. These same delays occurred whether at home or away. However, Infuse does support background audio playback.

## What We Tried

### CopyParty (Failed Solution)

I installed CopyParty Python web server on samba-storage-vm and set it up as a systemd service on port 3923. It was accessed via browser at `http://100.76.118.67:3923/`. Videos played perfectly on the laptop over Tailscale, but had inconsistent/failed playback on iPhone where WebM files mostly failed, and MP4 files were spotty. The root cause was inadequate setup - it should have used Docker plus Tailscale Serve for proper routing. CopyParty was completely removed. The critical flaw was that browser-based playback doesn't support background audio anyway.

### FE File Explorer Pro (Partial Solution)

I purchased FE File Explorer Pro app ($4.99) for iPhone and configured it with SMB (Windows Sharing) protocol, host `100.76.118.67` (Tailscale IP), username `mark`, Samba password, empty path, and share `storage` (auto-detected). Testing was done with Wi-Fi turned off on the phone to simulate being away from home. The result was fast performance with only slight delay on video start. The critical flaw is that it does not support background audio playback - the screen must stay on.

## Current State: UNSOLVED

Infuse has background audio but is unusably slow over Tailscale (45+45+60-second delays). FE File Explorer Pro is fast over Tailscale but has no background audio. The core problem remains unsolved: no app currently provides both fast streaming over Tailscale and background audio playback.
