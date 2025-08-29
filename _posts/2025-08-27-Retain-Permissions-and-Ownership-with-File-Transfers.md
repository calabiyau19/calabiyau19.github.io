---
layout: post
title: "How to transfer files on linux while preserving permissions and ownership"
draft: false
date:  2025-08-28
description: Here is a step-by-step procedure for transferring files from one Linux box (LPTHP) to your Docker VM (or any Linux system), preserving permissions and ownership, with all required commands explicitly stated, using the 'username username' (UID and GID = 1000) command. 
---


File Transfer: LPTHP to Docker VM (Preserving Permissions & Ownership)
Here is a step-by-step procedure for transferring files from LPTHP to your Docker VM (or any Linux system), preserving permissions and ownership, with all required commands explicitly stated, using the 'mark user' (UID and GID = 1000) command.
Goal
Copy files and folders from LPTHP (Linux Mint box) to docker-vm (the VM running your Docker containers).
Preserve ownership (mark:mark → UID:GID = 1000:1000)
Preserve permissions (e.g., rw-r--r--)
Correct any issues after the transfer if needed.
Step 1: Preparation on Both Machines
1.1 Ensure rsync is installed
Run on both LPTHP and docker-vm:

```sh
sudo apt update
```
```sh
sudo apt install -y rsync
```

1.2 Verify user mark exists and has UID/GID 1000

Run on docker-vm:
```sh
id mark
```

Expected output:

```sh
uid=1000(mark) gid=1000(mark) groups=1000(mark)
```

If UID/GID is not 1000, adjust your rsync or Docker configs.

Step 2: Transfer Files from LPTHP to docker-vm

Assume you're copying /home/mark/Documents/ProjectA from LPTHP to docker-vm.

2.1 Ensure target directory exists on destination

On docker-vm:
```sh
mkdir -p /home/mark/Documents/ProjectA
```
```sh
sudo chown -R mark:mark /home/mark/Documents/ProjectA
```

2.2 Perform the file transfer (from LPTHP)

On LPTHP:
```sh
rsync -avz -e ssh /home/mark/Documents/ProjectA/ mark@docker-vm:/home/mark/Documents/ProjectA/
```

Note the trailing slashes: /ProjectA/ to copy contents, not the directory itself.

Step 3: Verify on docker-vm

SSH into docker-vm:
```sh
ssh mark@docker-vm
```
```sh
ls -l /home/mark/Documents/ProjectA/
```

Expected output:
`
-rw-r--r-- 1 mark mark 1234 Aug 27 14:00 config.yaml
`

Step 4: Fix Ownership (if needed)

If any files are owned by root or another user:
```sh
sudo chown -R mark:mark /home/mark/Documents/ProjectA/
```

Or use UID/GID directly:
```sh
sudo chown -R 1000:1000 /home/mark/Documents/ProjectA/
```
Step 5: Fix Permissions (optional)

If files aren't writable:
```shj
chmod -R u+rwX /home/mark/Documents/ProjectA/
```

Explanation:

u+rwX: Adds read/write for user, execute only for directories or already executable files.

Optional Script on LPTHP

Save this as copy-to-docker-vm.sh:

```sh
#!/bin/bash
SOURCE_PATH="/home/mark/Documents/ProjectA/"
DEST_USER="mark"
DEST_HOST="docker-vm"
DEST_PATH="/home/mark/Documents/ProjectA/"

rsync -avz -e ssh "$SOURCE_PATH" "$DEST_USER@$DEST_HOST:$DEST_PATH"
```


Make it executable:
```sh
chmod +x copy-to-docker-vm.sh
```
```sh
./copy-to-docker-vm.sh
```

Summary

Task	Command

Install rsync	
```sh
sudo apt install -y rsync
```
Ensure target dir exists	
```sh
mkdir -p ... && sudo chown -R mark:mark ...
```
Transfer files	
```sh
rsync -avz -e ssh /src/dir/ mark@docker-vm:/dest/
```
Fix ownership (if needed)	
```sh
sudo chown -R mark:mark /dest/dir/
```
Fix permissions (if needed)	
```sh
chmod -R u+rwX /dest/dir/
```
Why Is sudo chown Included?

Although rsync -a does preserve ownership and permissions, this only works if:

Both users have matching UID/GID

rsync is not run with sudo on one side and not the other

Destination directory is writable

So chown is included as a safety step, in case of:

Mixed UIDs

Write errors

Root-owned files

Docker containers writing as root

TL;DR – When Is sudo chown Necessary?

Situation	Is chown needed?

Matching UID, using rsync -a as user	No

Transferring with mismatched UID or root	Yes

Destination permissions wrong / partial writes	Yes

Docker writes as root	Yes

As a post-check before use	Optional (common)

Final Takeaway

If:

You're logged in as mark on both machines

UID 1000 is consistent

You're not using sudo

Target is writable

Then:

You don’t need sudo chown. It’s included only as a remediation step.


---

Let me know if you’d like me to package and send these as downloadable `.txt` and `.md` files. I can also customize the filenames and add more formatting or structure if needed.
