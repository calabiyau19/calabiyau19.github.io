---
layout: post
title: "External Drive to Container Blueprint: Proxmox → VM → Docker"
draft: false
date: 2025-09-12
description: A repeatable blueprint for mounting external drives through Proxmox hosts to VMs and into Docker containers using NFS shares.
---

# Blueprint: External Drive → Proxmox → VM → Container

## 1. Attach and Identify the Drive (Proxmox host)

* Plug in the external drive.
* Run:

```sh
lsblk -f
```

* Identify the device (`/dev/sda1`, `/dev/sdb1`, etc.), filesystem, and UUID.

## 2. Create Mount Point(s) (Proxmox host)

* Decide where to mount it, e.g.:

```sh
sudo mkdir -p /mnt/Ext_T7_ssd_nuc3/audiobooks
sudo mkdir -p /mnt/Ext_T7_ssd_nuc3/podcasts
sudo mkdir -p /mnt/Ext_T7_ssd_nuc3/digitalbooks
```

* Mount it temporarily:

```sh
sudo mount /dev/sda1 /mnt/Ext_T7_ssd_nuc3
```

## 3. Make Mount Persistent with fstab (Proxmox host)

* Get the UUID:

```sh
blkid /dev/sda1
```

* Edit `/etc/fstab`:

```sh
sudo nano /etc/fstab
```

* Add line (replace UUID with yours):

```sh
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/Ext_T7_ssd_nuc3 ext4 defaults 0 2
```

* Test:

```sh
sudo mount -a
```

## 4. Export Folders via NFS (Proxmox host)

* Edit exports:

```sh
sudo nano /etc/exports
```

* Add lines for each library:

```sh
/mnt/Ext_T7_ssd_nuc3/audiobooks   192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash,insecure)
/mnt/Ext_T7_ssd_nuc3/podcasts     192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash,insecure)
/mnt/Ext_T7_ssd_nuc3/digitalbooks 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash,insecure)
```

* Apply:

```sh
sudo exportfs -ra
showmount -e localhost
```

## 5. Mount in VM (client side)

* Create mount points:

```sh
sudo mkdir -p /mnt/audiobooks /mnt/podcasts /mnt/digitalbooks
```

* Test mount:

```sh
sudo mount -t nfs 192.168.1.121:/mnt/Ext_T7_ssd_nuc3/audiobooks /mnt/audiobooks
sudo mount -t nfs 192.168.1.121:/mnt/Ext_T7_ssd_nuc3/digitalbooks /mnt/digitalbooks
sudo mount -t nfs 192.168.1.121:/mnt/Ext_T7_ssd_nuc3/podcasts     /mnt/podcasts
```

* Make persistent in `/etc/fstab` on VM:

```sh
192.168.1.121:/mnt/Ext_T7_ssd_nuc3/audiobooks   /mnt/audiobooks   nfs defaults 0 0
192.168.1.121:/mnt/Ext_T7_ssd_nuc3/podcasts     /mnt/podcasts     nfs defaults 0 0
192.168.1.121:/mnt/Ext_T7_ssd_nuc3/digitalbooks /mnt/digitalbooks nfs defaults 0 0
```

* Reload:

```sh
sudo mount -a
```

## 6. Map Folders into Containers (VM)

* In your `docker-compose.yml`:

```yaml
volumes:
  - ./config:/config
  - ./metadata:/metadata
  - /mnt/audiobooks:/audiobooks
  - /mnt/podcasts:/podcasts
  - /mnt/digitalbooks:/digitalbooks
```

* Restart container:

```sh
sudo docker compose down && sudo docker compose up -d
```

## 7. Add Library in the App

* In Audiobookshelf → add `/audiobooks` and `/podcasts`.
* In Calibre-Web → add `/digitalbooks`.

## ✅ Summary

With this process:

* New libraries (movies, ebooks, comics, etc.) just mean adding a new directory under `/mnt/Ext_T7_ssd_nuc3`, exporting it, mounting it in the VM, and mapping it into the container.
* Same steps every time → repeatable blueprint.