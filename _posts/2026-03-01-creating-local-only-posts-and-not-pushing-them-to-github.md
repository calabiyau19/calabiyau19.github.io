---
layout: post
draft: false
title: "Creating local only posts and not pushing them to Github."
date: 2026-03-01
description: "From time to time, I want to post something to the Pontifex site, but I only really want it to be local, sometimes because it contains sensitive information, other times it's just more a note than a post. This documents that process." 
---


## Publishing a Post Locally Only — Keeping It Off GitHub

### The Correct Workflow
Step 1 — Create your post file in _posts/ as normal.

Step 2 — Immediately open .gitignore in VS Code and add the filename before touching Git:
sh_posts/2026-03-01-your-new-post.md
Save it.

Step 3 — Stage only .gitignore:
shgit add .gitignore

Step 4 — Commit and push the .gitignore change:
shgit commit -m "Update gitignore - keep [post name] local only"
shgit push origin main

Step 5 — Build and verify locally:
shbundle exec jekyll build

Check your local site to confirm the post renders correctly.

#### Critical Rules

Always use git add .gitignore explicitly — never git add . for this workflow
Always get .gitignore committed and pushed before doing anything else with Git
The post file must never be touched by git add — once Git stages it, it will go to GitHub


##### Recovery — If You Accidentally Pushed the File First
Step 1 — Add the filename to .gitignore in VS Code and save.
Step 2 — Remove it from GitHub without deleting it locally:
```sh
git rm --cached _posts/2026-03-01-your-post.md
```

Step 3 — Stage, commit, and push:
```sh
git add .gitignore
```
```sh
git commit -m "Remove [post name] from GitHub, keep local only"
```

```sh
git push origin main
 ```
 