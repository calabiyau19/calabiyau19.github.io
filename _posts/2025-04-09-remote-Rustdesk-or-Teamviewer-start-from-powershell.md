---
layout: post
title: "Remotely start Rustdesk or Teamviewer from Powershell prompt"
draft: false
date:  2025-04-09
description: Short instructions to access a remote windows pc **IF you already have SSH access** but other GUI programs like Rustdesk or Teamviewer are not working.  This includes remote reboot instructions as well
---


**Remotely Start Rustdesk or TeamViewer / Reboot via SSH (Windows 11\)**

**1\. Switch to PowerShell from Command Prompt:**

```sh
powershell
```

**2\. Start Rustdesk (if installed in default path):**

```sh
Start-Process "C:\Program Files\RustDesk\RustDesk.exe"
```

**3\. Start TeamViewer (if installed in default path):**

```sh  
Start-Process "C:\Program Files (x86)\TeamViewer\TeamViewer.exe"
```

**4\. Search for Rustdesk or TeamViewer executables (if path is unknown):**

```sh  
Get-ChildItem -Path "C:\", "$env:LOCALAPPDATA" -Recurse -Include rustdesk.exe, TeamViewer.exe -ErrorAction SilentlyContinue
```

**5\. Reboot the remote Windows 11 machine:**

Option A – PowerShell-native command:

```sh
Restart-Computer -Force
```
Option B – Command-line equivalent:

```sh  
shutdown /r /t 0
```
