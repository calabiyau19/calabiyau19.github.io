---
layout: post
draft: false
Title: Upgrade Proxmox VE from version 8 to 9. 
date: 2026-05-18
description: "This post documents the steps to update both of my Proxmox mini PC servers from version 8 to 9, since version 8 will stop getting updates in the next two months. So this was a critical update I needed to do. It was more complicated than I thought, and it could not have been done successfully without some help. Luckily, I had some."

---


## Upgrading Proxmox VE 8 to 9 — What Actually Happened

This documents the real-world upgrade of two Proxmox VE 8.4 nodes to PVE 9.1, completed May 2026. NUC3 was upgraded first as the test node, NUC5 second as the critical production node. Both run standalone (no cluster). This is what was actually done, including mistakes and recoveries.

---

## System Details

| Node | IP | Boot Drive | NIC | Role |
|------|----|------------|-----|------|
| NUC3 (sv-proxmox-nuc3) | 192.168.1.121 | nvme0n1 | Intel (enp3s0) | Secondary |
| NUC5 (sv-proxmox) | 192.168.1.125 | nvme1n1 | RTL8125 r8125 DKMS (enp1s0) | Primary/Critical |

---

## What to Do Before Starting Either Node

**Backups first — non-negotiable.** Back up every VM on the node being upgraded before touching anything. On a monthly backup schedule, do not assume last month's backup is sufficient.

**Shut down all VMs** on the node being upgraded. Running VMs during a dist-upgrade risk corruption if something causes an unclean reboot mid-process.

**Verify the current state** before starting:

```sh
pveversion
```

Must show `8.4.x` or higher.

```sh
efibootmgr -v
```

This is the single most important pre-upgrade command. It shows exactly what is managing your boot process. Record the output. You need to know which entry is `BootCurrent` and that it points to `shimx64.efi` on your NVMe. This is your reference point if boot fails after upgrade.

---

## Pre-Upgrade Checklist

Run the official checklist tool and resolve every FAILURE before proceeding. WARNINGS must be understood but are not all blockers.

```sh
pve8to9 --full
```

**FAILURE: systemd-boot meta-package installed**

Remove it:

```sh
apt remove systemd-boot
```

Do not remove `systemd-boot-efi` unless the checklist specifically tells you to. On NUC3 only `systemd-boot` was present. On NUC5 both were present but only `systemd-boot` was flagged.

**NOTICE: LVM autoactivation**

Run the migration script:

```sh
/usr/share/pve-manager/migrations/pve-lvm-disable-autoactivation
```

**WARN: Pending package updates**

```sh
apt update && apt upgrade
```

**Important:** If this installs a new kernel version, DKMS will attempt to rebuild any third-party modules automatically. Watch for this line in the output:

```
dkms: autoinstall for kernel X.X.XX-XX-pve was skipped since the kernel headers for this kernel do not seem to be installed.
```

If you see it, the DKMS module was NOT rebuilt. Install the headers and rebuild manually before continuing:

```sh
apt install proxmox-headers-$(uname -r)
dkms install realtek-r8125/9.016.01 -k $(uname -r)
dkms status
```

Re-run `pve8to9 --full` after fixing each item until FAILURES = 0.

---

## NUC5 Only — Pin the Network Interface Name

NUC5 uses the RTL8125 NIC via the r8125 DKMS driver. The new PVE 9 kernel can rename network interfaces. If `enp1s0` gets renamed, `vmbr0` loses its physical uplink and all VMs lose network.

The `pve-network-interface-pinning` tool only exists in PVE 9. On PVE 8, create a systemd link file manually. Fish shell does not support heredoc syntax — use printf:

```sh
printf '[Match]\nMACAddress=Your-Nic-MAC-Address\n\n[Link]\nName=enp1s0\n' > /etc/systemd/network/10-enp1s0.link
```

Verify:

```sh
cat /etc/systemd/network/10-enp1s0.link
```

This takes effect on next boot. No reboot needed yet.

---

## Install tmux

Mandatory before running the upgrade. If SSH drops mid-upgrade without tmux, the dist-upgrade process dies and leaves the node in a partially upgraded state.

```sh
apt install tmux
tmux new -s upgrade
```

Confirm the status bar appears at the bottom before proceeding. If SSH drops during the upgrade, reconnect and run:

```sh
tmux attach -t upgrade
```

---

## Update APT Repositories

Switch Debian and Proxmox repos from bookworm to trixie. Leave Docker and Tailscale repos on bookworm — they are third-party and work fine on Trixie.

```sh
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
```

Check for a separate Proxmox no-subscription file — it may or may not exist depending on how the node was originally configured:

```sh
ls /etc/apt/sources.list.d/
```

If `pve-no-sub.list` exists:

```sh
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-no-sub.list
```

If the Proxmox repo is already in `sources.list` directly (as on NUC5), nothing extra is needed.

Refresh:

```sh
apt update
```

Expect 600-700+ packages available for upgrade. This is normal — it is the entire Debian base moving from Bookworm to Trixie, not routine daily updates.

**Stop immediately if apt update shows this error:**

```
401 Unauthorized — enterprise.proxmox.com
```

The upgrade creates a new file `/etc/apt/sources.list.d/pve-enterprise.sources` that activates the enterprise repo. Comment it out:

```sh
sed -i 's/^/# /' /etc/apt/sources.list.d/pve-enterprise.sources
```

Then re-run `apt update`.

---

## Run the Upgrade

Inside your tmux session:

```sh
apt dist-upgrade
```

**Config file prompts — answer as follows:**

- `apt-listchanges` output (OpenSSH changelog) → press `q`
- `/etc/issue` → **N**
- `/etc/lvm/lvm.conf` → **Y**
- `/etc/ssh/sshd_config` → **Y**
- `/etc/default/grub` → **N** on NUC5 (preserves `pcie_aspm=off`), **N** on NUC3 (no custom params but keeping current is safest)
- `libc6` restart services prompt → **Yes**
- `keyboard-configuration` (appeared on NUC5, not NUC3) → **Enter** to accept English (US)
- Any other prompt → press **D** to see the diff before deciding

**Stop immediately if you see:**

```
W: (pve-apt-hook) You are attempting to remove the meta-package 'proxmox-ve'!
```

Type `n`, find any remaining `bookworm` references in your sources, fix them, re-run `apt update`, then retry.

---

## Critical Steps Before Rebooting

This is the section that caused NUC3 to fail on the first attempt. Do not skip any of it.

**Step 1 — Reinstall GRUB to the EFI partition**

On NUC3 the EFI partition was not mounted — mount it first:

```sh
mount /dev/nvme0n1p2 /boot/efi
```

On NUC5 the EFI partition was already mounted at `/boot/efi` — skip the mount.

Then on both nodes:

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=proxmox
```

**Expected:** `Installation finished. No error reported.`

```sh
update-grub
```

**Expected:** First entry shows `Found linux image: /boot/vmlinuz-7.0.2-4-pve`

**Step 2 — Fix the removable bootloader path**

This updates the fallback EFI boot path that some firmware uses if the primary entry fails. It is what prevented NUC3 from having to go through two boot failures:

```sh
echo 'grub-efi-amd64 grub2/force_efi_extra_removable boolean true' | debconf-set-selections -v -u && apt install --reinstall grub-efi-amd64
```

**Step 3 — Verify pcie_aspm=off survived (NUC5 only)**

```sh
grep pcie_aspm /boot/grub/grub.cfg
```

**Expected:** `pcie_aspm=off` appears on the `7.0.2-4-pve` line. If missing — do not reboot. Edit `/etc/default/grub` to add it back and run `update-grub` again.

**Step 4 — Verify r8125 DKMS module for the new kernel (NUC5 only)**

```sh
dkms status
```

**Expected:** `realtek-r8125/9.016.01, 7.0.2-4-pve, x86_64: installed`

If the new kernel version is missing — do not reboot:

```sh
apt install proxmox-headers-7.0.2-4-pve
dkms install realtek-r8125/9.016.01 -k 7.0.2-4-pve
dkms status
```

---

## Reboot

Only after all steps above are verified:

```sh
reboot
```

From your management laptop, watch for the node:

```sh
ping 192.168.1.121   # NUC3
ping 192.168.1.125   # NUC5
```

Expected response within 60-90 seconds.

SSH in and verify:

```sh
pveversion
ip link show enp1s0   # NUC5 only
dkms status           # NUC5 only
```

---

## If the Node Does Not Come Back

**NUC3** has no Wi-Fi adapter — physical access required if the wired NIC fails.

**NUC5** has Wi-Fi at `192.168.1.126` and Tailscale at `100.96.107.82` as fallback access paths.

**GRUB boot failure recovery (what happened on NUC3):**

Boot from the Proxmox VE ISO USB stick. Select Advanced → Rescue Boot. At the shell:

```sh
vgchange -ay
mount /dev/mapper/pve-root /mnt
mount /dev/nvme1n1p2 /mnt/boot/efi   # adjust partition for your node
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys
mount --bind /proc /mnt/proc
mount --bind /run /mnt/run
mount -t efivarfs efivarfs /mnt/sys/firmware/efi/efivars
chroot /mnt
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=proxmox
update-grub
exit
reboot
```

Remove the USB stick before the reboot completes.

**NUC5 r8125 NIC missing after reboot:**

```sh
modprobe r8125
ip link show enp1s0
ifup enp1s0
```

If the module isn't found, headers are missing for the running kernel — install them and rebuild as described in the pre-upgrade section.

---

## Post-Upgrade Cleanup

**Disable the enterprise repo file** created during the upgrade on both nodes:

```sh
sed -i 's/^/# /' /etc/apt/sources.list.d/pve-enterprise.sources
```

**Install CPU microcode:**

NUC3 (Intel):

```sh
sed -i 's/main contrib/main contrib non-free-firmware/g' /etc/apt/sources.list
apt update && apt install intel-microcode
```

NUC5 (AMD) — `non-free-firmware` was already in sources.list:

```sh
apt update && apt install amd64-microcode
```

Reboot after microcode installation to load it.

**Suppress the subscription popup:**

```sh
sed -Ezi.bak "s/(function ?\(orig_cmd\) \{)/\1\n\torig_cmd\(\);\n\treturn;/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

This must be reapplied after any update that includes `proxmox-widget-toolkit`.

**Start VMs** one at a time and verify each has network before starting the next.

---

## What Went Wrong and Why

**NUC3 failed to boot after the first reboot attempt.** The GRUB EFI binary was not properly written to the EFI partition during the dist-upgrade. The node booted into BIOS repeatedly and had no network. Recovery required physical access, the Proxmox ISO USB stick, Rescue Boot, and manually running grub-install from a chroot. This was caused by not running the grub-install step before rebooting. It is now in the plan as a mandatory pre-reboot step.

**NUC5's wired NIC disappeared** the night before the upgrade during an unrelated hard reboot. A routine kernel update (`6.8.12-23-pve`) had installed weeks earlier but the r8125 DKMS module had not been rebuilt for it automatically because the kernel headers were not installed. The NIC only disappeared when the node booted into the new kernel for the first time. Recovery was done remotely via Tailscale over the Wi-Fi adapter.

**The enterprise repo file** (`pve-enterprise.sources`) is created automatically during the dist-upgrade on both nodes. It is not commented out by default and causes `401 Unauthorized` errors on every subsequent `apt update`. It must be manually disabled post-upgrade.

---

## What Was Actually Required vs Precautionary

**Required — would have failed without it:**
- Removing `systemd-boot` meta-package
- Running `update-grub` and `grub-install` before rebooting
- Rebuilding r8125 DKMS module for the new kernel before rebooting
- Disabling `pve-enterprise.sources` after upgrade

**Recommended — genuinely useful:**
- tmux session (SSH drops are real and would have been catastrophic mid-upgrade)
- Verifying `pcie_aspm=off` in grub.cfg before rebooting on NUC5
- The removable bootloader fix — cannot prove it prevented NUC5's boot failure but NUC5 booted cleanly first time unlike NUC3
- Interface pinning via systemd link file — the interface name did not change on NUC5 but the risk was real

**Precautionary but harmless:**
- Running `pve8to9 --full` multiple times between fixes — good practice, adds confidence
- LVM autoactivation migration — not strictly required for local-only storage but recommended by Proxmox
- Checking all sources.list.d files before upgrade — caught nothing on NUC5 but correct procedure

**Could have been skipped:**
- The `efibootmgr -v` and `lsblk -f` checks before starting — useful for understanding the system, but the upgrade would have proceeded the same way without them
