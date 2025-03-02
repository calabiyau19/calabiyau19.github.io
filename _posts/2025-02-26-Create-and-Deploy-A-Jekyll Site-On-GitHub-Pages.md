---
layout: post
title: "Creating and Deploying a Jekyll Site to GitHub Pages"
date:  2025-02-26
---


### **Prerequisites**

Before starting, ensure you have: 
**VS Code** installed and configured.  
**Remote SSH access** to your Linux system.  
**Jekyll installed** (`jekyll -v` should return a version number).  
**A GitHub account** for hosting the site.

---

### **Step 1: Open VS Code and Connect via SSH**

1. Open **VS Code**.  
2. Press **Ctrl \+ Shift \+ P** and select **"Remote-SSH: Connect to Host"**.  
3. Enter your SSH details and connect to your server.  
4. Once connected, open the **VS Code terminal** (**Ctrl \+ \`**) to begin.

---

### **Step 2: Navigate to Your Home Directory**

**Ensure you’re in the correct location before creating the project.**

cd \~  
pwd  \# Confirm the output is /home/your-username

**Expected Output:**

/home/your-username  
---

### **Step 3: Create a New Jekyll Site**

Run:

jekyll new my-tutorial-site

**Expected Outcome:**

* Jekyll creates `/home/your-username/my-tutorial-site/`.  
* Files like `_config.yml`, `_posts/`, `_layouts/`, etc., appear.

---

### **Step 4: Open the New Project in VS Code**

1. In VS Code, click **File \> Open Folder**.  
2. Select `my-tutorial-site`.  
3. The **Jekyll project structure** should be visible in the left sidebar.

---

### **Step 5: Fix `_config.yml` for GitHub Pages**

**Prevents broken links when hosted on GitHub Pages.**

1. Open `_config.yml` in VS Code.  
2. Add or modify these lines:

title: My Jekyll Tutorial Site  
description: A simple Jekyll site hosted on GitHub Pages  
baseurl: "/my-tutorial-site"  \# Must match GitHub repo name  
url: "https://your-github-username.github.io"  \# GitHub Pages URL

3. **Save the file (`Ctrl + S`)**.

---

### **Step 6: Start the Jekyll Server Locally**

Run:

bundle exec jekyll serve \--port 4001 \--baseurl ""

**Expected Output:**

* Jekyll starts on `http://127.0.0.1:4001/`.  
* The site should display correctly in your browser.

---

### **Step 7: Add a "Hello World" Page**

**Create a custom page for testing.**

touch hello-world.md

Open `hello-world.md` and add:

\---  
layout: default  
title: Hello World  
permalink: /hello-world/  
\---

\# Hello, World\!

Welcome to my Jekyll tutorial site.

**Save the file (`Ctrl + S`)**.

**Refresh `http://127.0.0.1:4001/` and verify "Hello World" appears in the navbar.**

---

### **Step 8: Initialize a Git Repository**

**Prepare to push the site to GitHub.**

git init  
git branch \-m main  \# Rename master to main (GitHub standard)

**Expected Output:**

Initialized empty Git repository in /home/your-username/my-tutorial-site/.git/  
---

### **Step 9: Add and Commit Files to Git**

**Stage and commit all project files.**

git add .  
git commit \-m "Initial Jekyll site"

**Expected Output:**

\[main (root-commit) abc1234\] Initial Jekyll site  
12 files changed, ...  
---

### **Step 10: Create a New Repository on GitHub**

1. Go to [GitHub](https://github.com/) and log in.  
2. Click **New Repository**.  
3. Set the **repository name** to `my-tutorial-site`.  
4. Choose **Public** and **do not initialize with a README**.  
5. Click **Create Repository**.

**GitHub displays a URL like:**

https://github.com/your-username/my-tutorial-site.git  
---

### **Step 11: Link GitHub Repository to Your Local Project**

**Connect your local Git repository to GitHub.**

Replace `your-username` with your actual GitHub username:

git remote add origin https://github.com/your-username/my-tutorial-site.git  
git remote \-v  \# Verify the connection

**Expected Output:**

origin  https://github.com/your-username/my-tutorial-site.git (fetch)  
origin  https://github.com/your-username/my-tutorial-site.git (push)  
---

### **Step 12: Push Your Site to GitHub**

**Upload your Jekyll site to GitHub Pages.**

git push \-u origin main

**Expected Output:**

Enumerating objects: 12, done.  
Counting objects: 100% (12/12), done.  
...  
To https://github.com/your-username/my-tutorial-site.git  
 \* \[new branch\]      main \-\> main  
---

### **Step 13: Enable GitHub Pages Hosting**

1. On GitHub, go to **Settings \> Pages**.  
2. Under **"Branch"**, select `main` and click **Save**.  
3. Wait a few minutes, then refresh the **Pages** section.

You should see:

Your site is live at https://your-username.github.io/my-tutorial-site/

**Click the "Visit Site" button to test.**

---

### **Step 14: Verify Your Site Works**

1. **Go to** `https://your-username.github.io/my-tutorial-site/`.  
2. **Test the links** (About, Hello World, Blog Post).  
3. If styles or links are broken, **clear your browser cache** or open in Incognito mode.

