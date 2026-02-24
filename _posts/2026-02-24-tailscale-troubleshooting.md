---
layout: post
draft: false
title: "LOCAL ONLY - Troubleshooting Tailscale when connection cannot be established to services"
date: 2026-02-24
description: "This document outlines, in order of most likely to succeed, the troubleshooting steps for resolving Tailscale connection issues between my laptop and Proxmox hypervisor servers. For example, today I discovered that system updates had modified kernel components required by Tailscale. Because I forgot to run my maintenance script—which notifies me when a reboot is necessary—my laptop wasn't rebooted after the update, causing Tailscale to malfunction."
---

## Tailscale Troubleshooting — When Services Won't Connect

<!-- Hero image suggestion: A split-screen graphic showing a browser spinning on a Tailscale URL on the left, and a terminal with `tailscale status` output on the right. Placement: just below the title. -->

The scenario: you click a shortcut to one of your services, the browser spins, nothing loads. The service has a Tailscale address (`100.x.x.x`) associated with it. Before diving deep, work through these steps in order — most Tailscale problems resolve in the first three.

---

## Check the Tailscale Admin Console First

Open your browser and go to [tailscale.com](https://tailscale.com) and log into your admin console. Look for two things: is your laptop showing online, and is the target VM showing online? This splits the problem immediately — you'll know whether it's your laptop, the remote node, or something in between before running a single command.

---

## Check If the Target VM Is Actually Running

If the target VM shows offline in the admin console, log into Proxmox and confirm it's powered on. It's easy to forget a VM was shut down when a service isn't in daily use.

---

## Check Tailscale Status on the Laptop

```sh
tailscale status
```

This confirms what the admin console showed and reveals whether your laptop itself is the offline node. Look for `offline` next to your laptop's hostname in the output.

---

## Try Bringing Tailscale Up

If your laptop shows offline, run:

```sh
tailscale up
```

Sometimes the daemon just needs a kick. Check status again after running it.

---

## Test the Target VM on Its Local Address

Try reaching the VM on its `192.168.x.x` address directly. If the service loads locally but not via Tailscale, the VM and service are healthy — Tailscale itself is the problem. If it doesn't load locally either, the VM or service is down regardless of Tailscale.

---

## Ping the Target via Its Tailscale IP

```sh
tailscale ping 100.x.x.x
```

Replace `100.x.x.x` with the Tailscale IP of the target VM from the admin console. This bypasses DNS and the application layer entirely and tests the raw tunnel. A timeout here confirms the tunnel is broken.

---

## Check the Tailscale Daemon Status

```sh
systemctl status tailscaled
```

Look specifically for this line in the output:

`Unable to connect to the Tailscale coordination server to synchronize`

That tells you the daemon is running but has lost contact with Tailscale's backend — a reboot is likely the fix.

---

## Check How Long Since the Last Reboot

```sh
uptime
```

If it's been more than a few days and system updates have run since the last reboot, stop here. A reboot is almost certainly all you need. Firmware, kernel, and low-level system package updates frequently leave services in a degraded state without triggering obvious failures elsewhere.

---

## Reboot the Laptop

```sh
sudo reboot
```

After linux-firmware, kernel, dbus, or initramfs updates, a clean reboot resolves issues that no amount of service restarts will fix. This is the right move before any deeper diagnostics when uptime is long and updates have run.

> **Note:** Do not restart `dbus` directly with `systemctl restart dbus` on a running desktop — it will immediately kill your graphical session and drop you to a black screen. A full reboot is always the safer path.

---

## Check Tailscale on the Target VM

If your laptop is confirmed online but still can't reach a specific VM, SSH into that VM using its local `192.168.x.x` address and check its Tailscale daemon:

```sh
systemctl status tailscaled
```

If it's in a failed or degraded state, restart it:

```sh
sudo systemctl restart tailscaled
```

---

## The Lesson

Your weekly update routine includes a reboot check at the end for good reason. Low-level updates — firmware, kernel, dbus, initramfs — need a reboot to take full effect. Tailscale is often the first service to show symptoms because it depends on stable system-level networking infrastructure. If the weekly reboot had run, none of this troubleshooting would have been necessary.
