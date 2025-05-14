---
layout: post
title: "Remote Startup of Rustdesk and Teamviewer services on Windows PC"
draft: false
date:  2025-05-14
description: How to remotely start up TeamViewer and/or Rust Desk services on a remote PC where both applications are already installed. 
---


How to remotely start up TeamViewer and/or Rust Desk services on a remote PC where both applications are already installed. This could also be used for other applications, and might happen if something else is running on the remote PC that has caused your application to stop running.    
This happened to me today when I tried to access a family member's remote PC to give them computer support and realized that both TeamViewer and RustDesk applications were not running.

Trying to start up the applications directly was tried, but this solution ended up being the fastest by far and the easiest to troubleshoot

**These commands are to be run *after* connecting over SSH to the remote Windows 11 PC.**

---

**1\. Check if RustDesk and TeamViewer are running:**


```sh

sc query rustdesk
sc query TeamViewer

```

*These commands check the current status (running, stopped, etc.) of the RustDesk and TeamViewer services.*

---

**2\. Start them if they are not running:**


```sh

sc start rustdesk 
sc start TeamViewer

```

*These commands start the RustDesk and TeamViewer services if they are installed and currently stopped.*

