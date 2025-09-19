---
layout: post
title: "Build a Better Bash with Atuin"
date:  2025-02-21
---
##Needs to be updated, so the Autocomplete works in Fish 


### **Step-by-Step Plan for Setting and Up Bash Completion and Atuin in New VMs**

This guide will ensure that every new Ubuntu Server VM you set up has **bash completion** for tab completion and **Atuin** for enhanced shell history.

---

## **Step 1: Update the System**

Before installing anything, update and upgrade all pickipackagesess.

```
sudo apt update && sudo apt upgrade -y
```

---

## **Step 2: Install Required Tools**

Since Ubuntu Server doesn’t include some essential tools by default, install the following:

bash  
enter  
`sudo apt install -y curl wget git unzip apt-transport-https ca-certificates lsb-release`

These tools are required for downloading and installing software, including Atuin.

---

## **Step 3: Install Bash Completion**

Bash completion allows for easier command entry via **tab completion**.

**Check if bash-completion is installed:**  
bash  
enter  
`dpkg -l | grep bash-completion`

1.   
   * If installed, move to Step 4\.

If not installed, install it:  
bash  
enter  
`sudo apt install -y bash-completion`

* 

**Ensure bash-completion is enabled** by verifying that this block exists in `~/.bashrc`:  
bash  
enter  
`if ! shopt -oq posix; then`  
  `if [ -f /usr/share/bash-completion/bash_completion ]; then`  
    `. /usr/share/bash-completion/bash_completion`  
  `elif [ -f /etc/bash_completion ]; then`  
    `. /etc/bash_completion`  
  `fi`  
`fi`

2. 

If missing, **add it to** `~/.bashrc`:  
bash  
enter  
`echo 'if ! shopt -oq posix; then`  
`if [ -f /usr/share/bash-completion/bash_completion ]; then`  
  `. /usr/share/bash-completion/bash_completion`  
`elif [ -f /etc/bash_completion ]; then`  
  `. /etc/bash_completion`  
`fi`  
`fi' >> ~/.bashrc`

* 

**Apply the changes:**  
bash  
enter  
`source ~/.bashrc`

3. 

---

## **Step 4: Install Atuin**

Atuin enhances shell history with powerful search and synchronization.

**Download and install Atuin using the official script:**  
bash  
enter  
`curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh`

1. 

**Ensure Atuin is added to your shell’s PATH** (so it works in all sessions):  
bash  
enter  
`source $HOME/.atuin/bin/env`

2. 

**Restart your shell** to finalize the setup:  
bash  
enter  
`exec "$SHELL"`

3. 

**Verify the installation:**  
bash  
enter  
`atuin --version`

4. If you see a version number, Atuin is correctly installed.

---

## **Step 5: Enable Atuin in Every SSH Session**

To make sure Atuin is automatically loaded in every session:

bash  
enter  
`echo 'source $HOME/.atuin/bin/env' >> ~/.bashrc`

Apply the changes:

bash  
enter  
`source ~/.bashrc`

---

## **Step 6: Test Everything**

### **Test Bash Completion**

Start typing a command, e.g.:  
bash  
enter  
`sudo apt    <<< add space also for more options`

1. Press `Tab` twice. If autocompletion works, you should see options like `install`, `update`, etc.

### **Test Atuin**

Run:  
bash  
enter  
`atuin search apt`

1. This should return previous commands containing "apt".  
2. Try **using Atuin autocomplete**:  
   * Start typing a previous command and press `Ctrl + R` to see enhanced history suggestions.

---

## **Step 7: Repeat for Every New VM**

Now, whenever you create a new VM, just follow this exact guide to set up **bash completion and Atuin** consistently across all machines.

---

### **Summary of Commands**

For reference, here’s the full list of commands to run in sequence:

bash  
enter  
`# Step 1: Update system`  
`sudo apt update && sudo apt upgrade -y`

`# Step 2: Install required tools`  
`sudo apt install -y curl wget git unzip apt-transport-https ca-certificates lsb-release`

`# Step 3: Install bash-completion (if needed)`  
`sudo apt install -y bash-completion`  
`echo 'if ! shopt -oq posix; then`  
`if [ -f /usr/share/bash-completion/bash_completion ]; then`  
  `. /usr/share/bash-completion/bash_completion`  
`elif [ -f /etc/bash_completion ]; then`  
  `. /etc/bash_completion`  
`fi`  
`fi' >> ~/.bashrc`  
`source ~/.bashrc`

`# Step 4: Install Atuin`  
`curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh`  
`source $HOME/.atuin/bin/env`  
`exec "$SHELL"`

`# Step 5: Make Atuin work in all future SSH sessions`  
`echo 'source $HOME/.atuin/bin/env' >> ~/.bashrc`  
`source ~/.bashrc`

`# Step 6: Verify everything`  
`atuin --version`  
`atuin search apt  # Test history search`

---

This plan is now **concise, correct, and repeatable** for every new VM you set up. 🚀 Let me know if you need any refinements\!

This setup primarily applies to **Bash** (the default shell for Ubuntu Server) and **Zsh**. However, Atuin also supports **Fish**, but requires a different configuration step.

### **Shell Compatibility:**

| Shell | Bash Completion | Atuin Support | Configuration Needed? |
| ----- | ----- | ----- | ----- |
| **Bash** (Default) | ✅ Yes | ✅ Yes | No extra config needed (already covered in the guide) |
| **Zsh** | ❌ No (Zsh has its own completion system) | ✅ Yes | Needs manual Atuin configuration |
| **Fish** | ❌ No | ✅ Yes | Uses a different config file (`env.fish`) |

---

## **If Using Zsh**

For Zsh users, Atuin still works, but Bash completion isn't needed. You **must** add Atuin’s environment config manually:

**Install Zsh (if not already installed)**  
bash  
enter  
`sudo apt install -y zsh`

1. 

**Add Atuin to your Zsh configuration**  
bash  
enter  
`echo 'source $HOME/.atuin/bin/env' >> ~/.zshrc`

2. 

**Apply the changes**  
bash  
enter  
`source ~/.zshrc`

3. 

---

## **If Using Fish**

Fish doesn’t use `.bashrc` or `.zshrc`, so Atuin needs a different approach:

**Add Atuin’s Fish configuration**  
bash  
enter  
`source $HOME/.atuin/bin/env.fish`

1. 

**Make it persistent**  
bash  
enter  
`echo 'source $HOME/.atuin/bin/env.fish' >> ~/.config/fish/config.fish`

2. 

**Restart Fish**  
bash  
enter  
`exec fish`

3. 

---

### **TL;DR**

* **Bash** → Works as is (guide is fine).  
* **Zsh** → Needs `source $HOME/.atuin/bin/env` in `~/.zshrc`.  
* **Fish** → Needs `source $HOME/.atuin/bin/env.fish` in `~/.config/fish/config.fish`.

Would you like help setting up Zsh or Fish instead of Bash? 🚀

**BUT WAIT THERE IS MORE**

You said:  
I have installed bash-completion and atuin on my proxmox, a Debian-based server. Can I add some basic upgrades to the base bash prompt that will be useful? I don't want to install Fish on all my servers since Bash is supported out of the box in all of them.

###### ChatGPT said:

Great call sticking with Bash—it keeps things consistent across all your servers without needing extra dependencies like Fish. Here are some practical Bash prompt upgrades that enhance usability, improve readability, and make working in the terminal more efficient.

---

## **1️⃣ Add Git Status to Your Prompt (If You Use Git)**

**If you work with Git repositories, showing the branch and status in your prompt is helpful. Add this to `~/.bashrc`:**

**bash**  
**enter**  
**`parse_git_branch() {`**  
  **`git branch 2>/dev/null | grep '*' | sed 's/* //'`**  
**`}`**

**`export PS1='\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[1;33m\] $(parse_git_branch)\[\e[0m\]\$ '`**

### **What This Does:**

**✅ Shows username and hostname in green**  
**✅ Shows the working directory in blue**  
**✅ Displays the current Git branch in yellow (if inside a Git repo)**

---

## **2️⃣ Show Exit Code of Last Command (Only if it Fails)**

**Sometimes, you run a command, and it fails—this makes it obvious by displaying the exit code in red.**

**Add this function to `~/.bashrc`:**

**bash**  
**enter**  
**`show_exit_code() {`**  
  **`EXIT_CODE=$?`**  
  **`if [ $EXIT_CODE -ne 0 ]; then`**  
    **`echo -e "\e[1;31mExit Code: $EXIT_CODE\e[0m"`**  
  **`fi`**  
**`}`**  
**`export PROMPT_COMMAND="show_exit_code; $PROMPT_COMMAND"`**

### **What This Does:**

**✅ If the last command fails, it will display:**  
**Exit Code: 1 (or whatever the exit code is)**  
**✅ If the last command succeeds, nothing changes**

---

## **3️⃣ Add Timestamps to Your History**

**Bash normally saves commands without timestamps. Enable timestamps so you can see when each command was run.**

**Add this to `~/.bashrc`:**

**bash**  
**enter**  
**`export HISTTIMEFORMAT="%F %T "`**

### **What This Does:**

**✅ `history` now shows timestamps like:**

**yaml**  
**enter**  
 **`1  2024-02-14 12:30:45 sudo apt update`**  
  **`2  2024-02-14 12:31:12 ls -la`**

---

## **4️⃣ Make Your Prompt Show the Last Command Execution Time**

**Ever run a command and wonder how long it took? This adds execution time info to your prompt.**

**Add this to `~/.bashrc`:**

**bash**  
**enter**  
**`export PROMPT_COMMAND='LAST_CMD_TIME=$SECONDS; PREV_LAST_CMD_TIME=$LAST_CMD_TIME; $PROMPT_COMMAND'`**  
**`export PS1='\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\] (\[\e[1;35m\]${PREV_LAST_CMD_TIME}s\[\e[0m\])\$ '`**

### **What This Does:**

**✅ Shows execution time of the last command in purple**  
**✅ If the last command took 5 seconds, prompt will look like:**

**ruby**  
**enter**  
**`user@server:~/folder (5s)$`**

**✅ If the command was instant, the (0s) disappears**

---

## **5️⃣ Enable Persistent History Across Sessions**

**By default, Bash only saves history at the end of a session, meaning if multiple terminals are open, they don’t share history. Fix this:**

**Add this to `~/.bashrc`:**

**bash**  
**enter**  
**`shopt -s histappend`**  
**`PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"`**

### **What This Does:**

**✅ Instantly saves each command you run to history**  
**✅ History is shared across multiple terminals (so all sessions see the latest commands)**  
**✅ Prevents history loss when multiple terminal windows are open**

---

## **6️⃣ Enable Better Navigation (Ctrl+Arrow for Word Jumps)**

**By default, Bash doesn’t let you jump word by word using `Ctrl+←` and `Ctrl+→` like in most modern shells.**

**Add this to `~/.inputrc` (create the file if it doesn't exist):**

**bash**  
**enter**  
**`"\e[1;5D": backward-word`**  
**`"\e[1;5C": forward-word`**

### **What This Does:**

**✅ Ctrl+← jumps back one word**  
**✅ Ctrl+→ jumps forward one word**  
**✅ Makes navigation faster when editing long commands**

---

## **7️⃣ Shorten Your Prompt on Long Paths**

**By default, if you're deep inside a directory, your prompt gets very long. Fix this by shortening the path.**

**Add this function to `~/.bashrc`:**

**bash**  
**enter**  
**`export PS1='\u@\h:\w$( [ ${#PWD} -gt 40 ] && echo "…${PWD: -20}")\$ '`**

### **What This Does:**

**✅ Shortens the displayed path if it's longer than 40 characters**  
**✅ Example:**

**ruby**  
**enter**  
**`user@server:/home/user/projects/super-long-directory-name$ # Normal`**  
**`user@server:…per-long-directory-name$ # Shortened`**

---

## **8️⃣ Improve "cd" Behavior (Auto Push to Stack)**

**Make `cd -` work like a stack so you can jump back to previous directories easily.**

**Add this to `~/.bashrc`:**

**bash**  
**enter**  
**`cd() {`**  
  **`builtin cd "$@" || return`**  
  **`pwd > ~/.lastdir`**  
**`}`**  
**`alias -- -='cd "$(cat ~/.lastdir)"'`**

### **What This Does:**

**✅ Every time you `cd` into a directory, it remembers the previous one**  
**✅ Typing `-` jumps back to the last directory**  
**✅ Example:**

**bash**  
**enter**  
**`cd /var/log`**  
**`cd /etc`**  
**`-`**  
**`# Now you're back in /var/log!`**

---

## **Final Steps: Apply the Changes**

**After making changes to `~/.bashrc`, apply them with:**

**bash**  
**enter**  
**`source ~/.bashrc`**

**If you edited `~/.inputrc`, apply changes with:**

**bash**  
**enter**  
**`bind -f ~/.inputrc`**

---

### **📌 TL;DR \- What These Enhancements Do**

**✔ Adds Git branch status to your prompt**  
**✔ Shows exit codes for failed commands**  
**✔ Saves timestamps in history**  
**✔ Displays execution time for commands**  
**✔ Keeps Bash history persistent across sessions**  
**✔ Enables Ctrl+← / Ctrl+→ word jumps**  
**✔ Shortens long paths in the prompt**  
**✔ Improves `cd -` navigation**

---

**These upgrades keep Bash lightweight but powerful, giving you some of Fish/Zsh's best features without adding new dependencies.**

**Would you like a specific feature customized further? 🚀**

##### **You said:**

**this only keeps them current for the current session though right?**

###### **ChatGPT said:**

**You're correct—if you add these changes to `~/.bashrc` and just run `source ~/.bashrc`, they'll only apply to the current session. However, once they are saved in `~/.bashrc`, they persist across reboots and new SSH sessions automatically.**

**To ensure all changes always apply in every new terminal session, just do this once:**

1. **Make sure everything is in `~/.bashrc`**  
   **If you’ve added the changes to `~/.bashrc`, they will persist across all future SSH sessions and logins. You don’t need to manually `source ~/.bashrc` every time—you just need to restart your shell or open a new terminal.**

**For Immediate Effect in Current Session**  
**After editing `~/.bashrc`, run:**  
**bash**  
**enter**  
**`source ~/.bashrc`**

2. **This ensures the changes take effect immediately without needing to restart your shell.**  
3. **For Changes in All New Terminals or SSH Sessions**  
   **The next time you open a new terminal or SSH into the server, these settings will already be applied. You do not need to run anything again.**

---

### **Exceptions Where It Won’t Persist**

1. **If you edit only `~/.bash_profile` instead of `~/.bashrc`**  
   * **`~/.bash_profile` runs only for login shells (like when logging in via SSH), but not for interactive shells (like opening a new terminal from an already logged-in session).**  
   * **Solution: Always put changes in `~/.bashrc` for consistency.**  
2. **If you use `su -` instead of `su`**  
   * **`su -` loads a clean environment without user-specific configs.**

**Solution: If switching users, explicitly source the file:**  
**bash**  
**enter**  
**`source ~/.bashrc`**

*   
3. **If your shell is not Bash**  
   * **If you’re using Zsh or Fish, `~/.bashrc` won’t be loaded.**  
   * **Solution: Modify `~/.zshrc` or `~/.config/fish/config.fish` instead.**

---

### **To Apply Changes System-Wide (All Users)**

**If you want these prompt improvements for all users on the system, instead of just your account:**

**Edit the global Bash configuration:**  
**bash**  
**enter**  
**`sudo nano /etc/bash.bashrc`**

1.   
2. **Add the same prompt modifications inside that file.**  
3. **Save and exit (`Ctrl+X`, then `Y`, then `Enter`).**

**Apply changes immediately:**  
**bash**  
**enter**  
**`source /etc/bash.bashrc`**

4. 

---

### **TL;DR**

* **If added to `~/.bashrc`, changes persist for your user across reboots and SSH sessions.**  
* **If added to `/etc/bash.bashrc`, changes persist system-wide for all users.**  
* **After editing, `source ~/.bashrc` makes changes apply immediately without restarting.**

**Would you like help setting this up across multiple servers at once? 🚀**

##### **You said:**

**how do I add the first one for example to \~/.bashrc and make it persistent?**

###### **ChatGPT said:**

**Great question\! Let's go step by step to add the Git branch status to your Bash prompt and make sure it persists across all future sessions.**

---

### **Step 1: Open `~/.bashrc` for Editing**

**Run the following command to open your Bash configuration file in `nano`:**

**bash**  
**enter**  
**`nano ~/.bashrc`**

---

### **Step 2: Add the Git Branch Function to Your Prompt**

**Scroll to the bottom of the file (using the arrow keys) and add this code:**

**bash**  
**enter**  
**`# Function to show the current Git branch`**  
**`parse_git_branch() {`**  
  **`git branch 2>/dev/null | grep '*' | sed 's/* //'`**  
**`}`**

**`# Custom Bash Prompt with Git Branch`**  
**`export PS1='\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[1;33m\] $(parse_git_branch)\[\e[0m\]\$ '`**

---

### **Step 3: Save the File and Exit**

1. **Press `CTRL + X` to exit `nano`.**  
2. **Press `Y` to confirm saving changes.**  
3. **Press `Enter` to save and exit.**

---

### **Step 4: Apply the Changes Immediately**

**Run this command to reload the `~/.bashrc` file without logging out:**

**bash**  
**enter**  
**`source ~/.bashrc`**

---

### **Step 5: Verify That It Works**

**Navigate to a directory that contains a Git repository:**  
**bash**  
**enter**  
**`cd /path/to/git-repo`**

1. 

**Check if the Git branch name appears in your prompt. If you're inside a Git repo, it should now display:**  
**ruby**  
**enter**  
**`user@server:/path/to/repo main$`**

2. **Where `main` (or `master`) is the branch name.**

---

### **Step 6: Make Sure It Works in New SSH Sessions**

* **Close your terminal and open a new one.**  
* **SSH back into your server and check if the Git branch appears.**  
* **If it does, your custom Bash prompt persists across all sessions\!**

---

### **Bonus: Make It System-Wide (All Users)**

**If you want all users on your server to have this Git-enabled prompt, edit the global Bash config:**

**bash**  
**enter**  
**`sudo nano /etc/bash.bashrc`**

**Then, add the same function and PS1 line to the bottom of the file. Save and exit, then apply:**

**bash**  
** **  
**`source /etc/bash.bashrc`**

**Now, every user on the system will have this prompt\!**

---

### **You're Done\! 🎉**

**This Git-enhanced Bash prompt will now persist across reboots and all SSH sessions.**

**Would you like to tweak it further? Maybe add colors or execution time? 🚀**


