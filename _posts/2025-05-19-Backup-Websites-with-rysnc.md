---
layout: post
title: "Backup websites with rsync across the network"
draft: false
date:  2025-05-19
description: Simple documentation for backing up the Jekyll and Hugo websites being developed on my local test server that are eventually hosted on GitHub.   
---


**\*\*Premise.\*\***  

I wanted to learn about rsync, a terminal-based backup tool. It is used by a lot of linux programs like Deja Dup.  Additionally, I was seeking a straightforward method to back up the websites I develop locally before pushing them to GitHub for publication and viewing.    
While I could have simply copied and pasted the files across my network, terminal solutions are simple, elegant, and easily recalled (with terminal history like Atuin). Once you become familiar with them, they work efficiently and usually quicker than GUI methods.  

My websites are static site generators, either Jekyll-based or Hugo-based. They reside on an old laptop that I installed XUbuntu on, and that's pretty much all I have running on it besides a self-hosted resume maker. I've been hearing a lot about rsync but never used it since I came from Windows and was used to either backups or just copying and pasting folders and files I wanted to preserve in a secondary location.  
I thought this was a great opportunity to learn about rsync. 

**Step one:**

Decide where you want to store the copies and create their locations.  My “backup” location is on a second partition on my main SSD and is called Linux\_Extra with a folder called Website-Backups that houses subfolders, one for Hugo and one for Jekyll sites.  

**Step two:**

Make sure you understand the format of the rsync command. Whether you want to copy the entire folder or just the files within the folder affects the formatting of the command. After the word rsync, the \-av doesn't really do anything other than only back up changed folders or changed files if you keep running it over and over. Then you get into the part of the command that tells you what the source is, which is the first part, and what the destination is, which is the second part. 

**Step three:**

The commands.  The first is the rsync command using generic placeholders for the source and destination locations.     

Generic:    

```sh    
rsync \-av username@host\_or\_ip:/full/path/to/source-folder /local/path/to/backup/destination/  
  
```


Actual:  

```sh
rsync \-av mark@lpt-dell:/home/mark/calabiyau19.github.io /media/Linux\_Extra/Website-Backups/Jekyll/   

rsync \-av mark@192.xxx.x.xxx:/home/mark/calabiyau19.github.io /media/Linux\_Extra/Website-Backups/Jekyll/     

```

The source:  mark@lpt-dell:/home/mark/[calabiyau19.github.io](http://calabiyau19.github.io)    NOTE: the single space after this.    
The destination:  /media/Linux\_Extra/Website-Backups/Jekyll/   

Other actual examples:   
```sh

rsync \-av mark@hostnamel:/home/mark/Desktop/demosite /media/Linux\_Extra/Website-Backups/Jekyll/   

rsync \-av mark@192.xx.x.xxx:/home/mark/my-doc-website /media/Linux\_Extra/Website-Backups/Hugo/  

rsync \-av \--dry-run mark@192.xxx.x.xxx:/home/mark/my-doc-website /media/Linux\_Extra/Website-Backups/Hugo/     
  
```

**Step four:**  
Flags \- options

Common Flags  
\-a          "Archive" mode — preserves permissions, timestamps, symlinks, etc.     
\-v          "Verbose" — prints progress to the terminal    
\-z          (optional) Compress data in transit (useful over slow links)    
\-P          (optional) Show progress \+ partial transfers for big files    
\--delete    (⚠️ optional) Deletes files from the destination that were deleted from the source    
\--dry-run   (super useful) Preview what will happen without making any changes    
\-e ssh      (optional) Forces rsync to use SSH (default for remote hosts anyway)  

   
