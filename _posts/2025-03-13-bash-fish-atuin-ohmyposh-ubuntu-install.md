---
layout: post
title: "Bash Fish Tide Atuin Oh My Posh"
draft: true
date:  2025-03-13
description:  Comparison between the various features of Bash, Fish, Atuin, Oh, My Posh, Tide, plus prompt completion and prompt styles
---




After installing Ubuntu Server, whichever version you chose, the first thing you need to do after logging in via SSH is run the update commands.

```sh
sudo apt update && sudo apt upgrade -y 
```
and then you reboot when done. 

---
## **BASH**


Step 2: Verify Bash is installed or install if needed


```sh
command -v bash
```
This command will indicate whether bash is installed on your server. In my case, it was, which is normal for a regular Ubuntu install.  

If you need to install it, you can do so with the following command. Once installed, verify it using the next command:
```sh
sudo apt install -y bash
```
```sh
bash --version
```
NOTE:  At this point, bash and bash-completion, which is where you type a bash command like ‘sudo apt [space]’ and then ‘[tab] [tab]’ and you will see options for the next word or words you can add to sudo apt such as update or upgrade, etc. 


Step 3: Add Bash Auto-Suggestions

NOTE: This has been more helpful to me than the basic bash [tab][tab] completion because it automatically displays old commands you previously entered as you start typing a new command. Then, you can hit the right arrow on your keyboard, completing the rest of the command. This process quickly and accurately finishes longer or repetitive commands. 


**UPDATE & CORRECTION:**

Bash does not have an add-on for adding auto-suggestion to the prompt like Fish does. 

---
## **FISH**

Step 1:  Update server
```sh
sudo apt update
```
Step 2: Install Fish
```sh
sudo apt install -y fish
```
Step 3:  Verify installation
```sh
fish --version
```
should return fish, version 3.x.x

Step 4: Start Fish terminal session
```sh
fish
```
will see a new prompt, slightly different.  Colors different
mark@test-vm ~> 

  
 NOTE: The next command is not required, but you can run it to see which shells exist on the system that you can change without adding them. These are the shells installed by default in Ubuntu server 24.04 that you can switch to.   

```sh
cat /etc/shells
```
Will list Bash, Fish, Tmux, etc.  All we care about is that Fish is there.  

Step 5: Make Fish default shell
```sh
chsh -s /usr/bin/fish
```
It will prompt for your password because you are making changes to the user settings.

NOTE: This only changes the shell for you, not system-wide.

Step 6:  Begin a new session to start up the Fish shell
```sh
exec fish
```
  
---
## **OH MY POSH**

Step 1: Update curl and client certificates

```sh
sudo apt update && sudo apt install --reinstall ca-certificates curl -y
```
Step 2: Install Unzip - needed for installation
```sh
sudo apt install -y unzip
```

Step 3: Install Oh My Posh for Linux
```sh
curl -s https://ohmyposh.dev/install.sh | bash -s
```
Step 4: Add Oh My Posh to $PATH
```sh
export PATH=$PATH:/home/mark/.local/bin
```
Step 5: Add Oh My Posh to Your Fish Configuration
```sh
set -Ux fish_user_paths /home/mark/.local/bin $fish_user_paths
```
Step 6:  Verify installation
```sh
oh-my-posh --version
```
Returns something like 25.x.x if successful

---
## **PROMPT MODIFICATION**

Step 1: Download Nerd Fonts

```sh
mkdir -p ~/.local/share/fonts && cd ~/.local/share/fonts
wget https://github.com/ryanoasis/nerd-fonts/releases/latest/download/FiraCode.zip
```
Makes local directory ~/.local/share/fonts and downloads the nerd fonts.

Step 2. Extract the font files
```sh
unzip FiraCode.zip
rm FiraCode.zip
```
Step 3. Install fontconfig
```sh
sudo apt install -y fontconfig
```
Step 4. Refresh the fonts cache
```sh
fc-cache: succeeded
```
Step 5. Set the Nerd Font in GNOME Terminal  

Open GNOME Terminal settings:  

-Click on Edit → Preferences  

-Select your active profile - mine was Everforest Dark Hard  

Change the font:  

-Find the "Text Appearance" section.  

-Click on the "Custom Font" checkbox.  

-Select FiraCode Nerd Font (I did not select the mono one)  

Save and restart GNOME Terminal.  

Reconnect to your Ubuntu Server via SSH.


Step 6. Test if Nerd Fonts are working
```sh
oh-my-posh print primary
```
You should see something like this:

[insert prompt image here]


Step 7:  Choose a Theme from those downloaded with Oh My Posh
```sh
ls ~/.cache/oh-my-posh/themes/
```
Preview Themes at:
https://ohmyposh.dev/docs/themes  - or - https://github.com/JanDeDobbeleer/oh-my-posh/tree/main/themes (up to date)

Step 8: Test theme temporarily
 
```sh
oh-my-posh init fish --config ~/.cache/oh-my-posh/themes/theme-name.omp.json | source
```
Replace ***theme-name*** with one you chose

Step 9:  Make Permanent
```sh
echo 'oh-my-posh init fish --config ~/.cache/oh-my-posh/themes/theme-name.omp.json | source' >> ~/.config/fish/config.fish
```
Replace ***theme-name*** with one you chose


PROMPT MODIFICATION with Tide using the Fisher plugin for Fish

Step 1: Install Fisher (the Fish Plugin Manager)
```sh
curl -sL https://git.io/fisher | source && fisher install jorgebucaran/fisher
```

Step 2: Install Tide Prompt for Fish
```sh
fisher install IlanCosman/tide
```
Step 3:  Configure Tide
```sh
tide configure
```
This runs the Q&A wizard to set the prompt based on the answers you give to the questions

---
## **Install Atuin for added history search functionality**

Step 1:  Install Required Dependencies
```sh
sudo apt install curl git -y
```

Step 2: Run Atuin installation script
```sh
curl -sSL https://raw.githubusercontent.com/ellie/atuin/main/install.sh | bash
```

Step 3: Add Atuin to your PATH. This is a Fish-specific command, different from Bash or Zsh.
```sh
source $HOME/.atuin/bin/env.fish
```

Step 4:  Test installation
```sh
atuin --version
```
Step 5:  Make PATH change permanent
```sh
echo 'source $HOME/.atuin/bin/env.fish' >> ~/.config/fish/config.fish
```
HERE is the correct CONFIG.FISH & PATH for this setup.

'if status is-interactive"  
atuin init fish | source  
end  
set -Ux PATH /usr/local/bin /usr/bin /bin $HOME/.local/bin  
source $HOME/.atuin/bin/env.fish

Run:
```sh
nano ~/.config/fish/config.fish
```
And make sure config.fish is the same as the above

---

NOTE: For Bash - add the following to make Atuin persistent

```sh
source $HOME/.atuin/bin/env
```
```sh
echo 'source $HOME/.atuin/bin/env' >> ~/.bashrc
```


Summary: After working with Bash, changing to Fish, adding Oh My Posh and Tide to make custom prompts, and finally adding Atuin for Auto Suggestion—which is the key for an easy terminal experience — here are my preferences.

FISH + TIDE + ATUIN     AND drop Oh My Posh.  Nice but not a must-have

UPDATE:  In a hurry?  Just install Atuin in the normally default Bash shell









