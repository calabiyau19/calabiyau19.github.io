---
layout: post
title: "How to identify the method by which a Linux application was installed"
draft: false
date:  2025-08-13
description: Sometimes you do not install apps via the software manager or remember off hand how you did install a specific app.  This article shows a few different ways to remove unwanted software when you are not sure of the installation method.  Done on a linux mint (ubuntu) system. 
---


How to identify the method by which a Linux application was installed 08132025

**Option A:  Check and see if it was installed via APT**

```sh
dpkg \-l | grep \-i phoenix
```

If you see something like:

```sh
`ii  phoenix-code  1.2.3  all  Some description here`  
```` 
…then it was installed via APT, and you can uninstall it with:

```sh
`sudo apt remove --purge phoenix-code`  
```

**Option B: Check if it was installed as a Flatpak**

```sh
flatpak list | grep \-i phoenix 

```


If it appears here, uninstall with:

```sh 
`flatpak uninstall <app-id>`  
```


To list all details, you can run:

```sh
`flatpak list --app`  
```
 
**Option C: Check if it was installed as a Snap (unlikely on Mint unless Snap was re-enabled)**
```sh
snap list | grep \-i phoenix  
```


If it shows up:

```sh
sudo snap remove \<package-name\>  
```


**Option D: Check if it’s just an AppImage or a manually extracted binary**

```sh
ls \~/Downloads  
ls \~/Applications  
ls \~/.local/share/applications/  
```


Example output:  I was looking for Phoenix code

PhoenixCode.desktop

Then a FIND command like this:
```sh
find \~ \-type d \-iname "\*phoenix\*" 2\>/dev/null        
```
 
Output: 

```sh
/home/mark/Documents/Phoenix Code  
/home/mark/.phoenix-code  
/home/mark/.phoenix-code/src-node/www/phoenix-splash  
/home/mark/.local/share/phoenix-code.app  
```

### **Step 1: Remove the Menu Entry (if you haven’t yet)**

You already found the shortcut file:

```sh  
`~/.local/share/applications/PhoenixCode.desktop`  
```


To delete it:

```sh
`rm ~/.local/share/applications/PhoenixCode.desktop`  
```

That removes it from your app menu.  
OR JUST GO TO THE MENU AND RIGHT CLICK \- UNINSTALL

**✅ Step 2: Delete the App Itself**

Now, delete the actual app and its files.

You can do this in three separate commands (each one deletes one folder):

```sh

`rm -rf ~/Documents/Phoenix\ Code`  
`rm -rf ~/.phoenix-code`  
`rm -rf ~/.local/share/phoenix-code.app`  
```

**Note:** The `\` in `Phoenix\ Code` escapes the space in the folder name

### **Step 3 (Optional): Double-check It's Gone**

You can check again if anything related is still hanging around:

```sh

`find ~ -iname "*phoenix*" 2>/dev/null`

```

Delete anything you find \- and be careful it IS for the app you want to remove.

