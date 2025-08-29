---
layout: post
title: "Git Command Reference Sheet"
draft: false
date:  2025-08-28
description: Most used Git commands to help maintain and update your GitHub repo.  Git tracks changes not files.  Most operations are reversible if you understand the process
---



# Git Commands Reference Guide

A comprehensive reference for essential Git terminal commands organized by category.

## Basic Workflow
```bash
git init                    # Initialize a new Git repository
git clone <url>            # Clone a remote repository
git add <file>             # Stage specific file
git add .                  # Stage all changes
git commit -m "message"    # Commit staged changes
git push                   # Push to remote repository
git pull                   # Fetch and merge from remote
```

## Checking Status & History
```bash
git status                 # Show working directory status
git log                    # Show commit history
git log --oneline          # Compact commit history
git log --graph            # Show branch/merge history graphically
git show <commit>          # Show details of specific commit
git diff                   # Show unstaged changes
git diff --staged          # Show staged changes
git diff <commit>          # Compare with specific commit
```

## Branch Management
```bash
git branch                 # List local branches
git branch -a              # List all branches (local + remote)
git branch <name>          # Create new branch
git checkout <branch>      # Switch to branch
git checkout -b <branch>   # Create and switch to new branch
git switch <branch>        # Modern way to switch branches
git merge <branch>         # Merge branch into current branch
git branch -d <branch>     # Delete branch (safe)
git branch -D <branch>     # Force delete branch
```

## Remote Repository Management
```bash
git remote -v              # Show remote repositories
git remote add <name> <url> # Add remote repository
git fetch                  # Download changes without merging
git push -u origin <branch> # Push and set upstream branch
git push origin --delete <branch> # Delete remote branch
```

## Undoing Changes
```bash
git restore <file>         # Discard unstaged changes (modern)
git checkout -- <file>     # Discard unstaged changes (legacy)
git restore --staged <file> # Unstage file (modern)
git reset HEAD <file>      # Unstage file (legacy)
git commit --amend         # Modify last commit
git reset --soft HEAD~1    # Undo last commit, keep changes staged
git reset --hard HEAD~1    # Undo last commit, discard changes
git revert <commit>        # Create new commit that undoes changes
```

## Stashing (Temporary Storage)
```bash
git stash                  # Temporarily save uncommitted changes
git stash push -m "message" # Stash with description
git stash list             # Show all stashes
git stash pop              # Apply and remove most recent stash
git stash apply            # Apply stash without removing it
git stash drop             # Delete specific stash
git stash clear            # Delete all stashes
```

## Advanced Operations
```bash
git rebase <branch>        # Reapply commits on top of another branch
git rebase -i HEAD~3       # Interactive rebase (edit last 3 commits)
git cherry-pick <commit>   # Apply specific commit to current branch
git tag <tagname>          # Create lightweight tag
git tag -a <tag> -m "msg"  # Create annotated tag
git blame <file>           # Show who changed each line
git bisect start           # Start binary search for bug
```

## Configuration
```bash
git config --global user.name "Name"     # Set global username
git config --global user.email "email"   # Set global email
git config --list                        # Show all config settings
git config --global init.defaultBranch main # Set default branch name
```

## Useful Aliases (add to your config)
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## Quick Commit & Push Workflow
```bash
git add .
git status
git diff --staged
git commit -m "your message"
git push
```

## Most Important Commands to Master First

1. **`git status`** - Always know where you stand
2. **`git log --oneline`** - Understand your history
3. **`git diff`** and **`git diff --staged`** - Review changes
4. **`git stash`** / **`git stash pop`** - Handle interruptions
5. **`git checkout -b <branch>`** - Work on features safely
6. **`git merge`** - Integrate changes
7. **`git pull`** - Stay synchronized
8. **`git reset`** - Fix mistakes safely

## Key Concepts

- Git tracks changes, not files
- Most operations are reversible if you understand the commands
- The staging area lets you control exactly what gets committed
- Branches are lightweight and encourage experimentation
- Stashing helps manage context switching in development work