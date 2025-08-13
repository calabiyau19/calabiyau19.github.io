---
layout: post
title: "How to make Docker data persistent across reboots"
draft: false
date:  2025-08-09
description: By default when a Docker app is spun up the data for it only exists for as long as the container(s) are up and running.  The data is lost if the container goes down or is rebooted for any reason.  THIS process stores the data on the host machine running the Docker containers so if the containers are restarted, the data is pulled from where it is stored on the host machine.   
---

How to make Docker data persistent:
The new container best practice is to create folders on the host to store the app’s data.
This assumes you are using docker compose YAML files and docker run commands

```sh
mkdir -p ~/myapp/data  # generic example
```

With docker-compose.yml:

```yaml
services:
  myapp:  # generic example
    image: myimage:latest   #generic example
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/myapp   # <- app’s data dir inside container.  ./ = current directory - home, in this case
```

Bring up the containers.

```sh
docker compose up -d
```

Verify it’s persistent - using [adguardhome] as the container name

{% raw %}

```sh
docker inspect adguardhome --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'

```
{% endraw %}

You should see your host path mapped to the container path.

Existing container (already running, but NOT persistent yet)
Create host folders:

```sh
mkdir -p ~/myapp/data  # generic example
```

Example: Using Adguard as the example.
```sh
mkdir -p ~/adguard-home/work
mkdir -p ~/adguard-home/conf
```

Copy data out of the running container (so you don’t lose current settings):

```sh
docker cp myapp:/var/lib/myapp ~/myapp/  # generic example
```

Example: Using Adguard as the example.
```sh
docker cp adguardhome:/opt/adguardhome/work ~/adguard-home/
docker cp adguardhome:/opt/adguardhome/conf ~/adguard-home/
sudo chown -R $USER:$USER ~/adguard-home/work ~/adguard-home/conf
```


Edit docker-compose.yml to add the volumes:
```yaml
volumes:
  - ./work:/opt/adguardhome/work
  - ./conf:/opt/adguardhome/conf
```

Then:

```sh

cd ~/adguard-home
docker compose down
docker compose up -d

```

Verify mounts (same inspect command as above).

{% raw %}
```sh
docker inspect adguardhome --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
```
{% endraw %}

Notes you’ll need exactly once
Pick the right container path: every app has a data dir; check its docs or container README.
(AdGuard: /opt/adguardhome/work and /opt/adguardhome/conf.)

Permissions: if you hit “permission denied” when backing up or copying, fix ownership on the host:

```sh
sudo chown -R $USER:$USER ~/myapp
```

Backups: once persistent, back up the host folders (e.g., with your rsync command).

That’s it. New or existing, the core idea is the same: bind‑mount a host folder into the container’s data directory so the data lives outside the container and survives rebuilds/reboots.


