---
title: Tracking & Syncing my dotfiles!
date: 2025-06-22
description: .dotfiles or... (.)files²?
toc: true
math: true
draft: false
categories: linux
tags:
  - chezmoi
  - arch
  - git
  - dotfiles
  - config
---

*.dotfiles or... (.)²files?*

## - Using Git + Github, & tracking dotfiles with an alias.
- ... as mentioned on the [ever-wise *Arch Wiki*.](https://wiki.archlinux.org/title/Dotfiles#Tracking_dotfiles_directly_with_Git)

``` bash
# 1. Create a bare Git repo to track dotfiles
git init --bare ~/.dotfiles

# 2. Create an alias to simplify dotfiles management.
# Tells (/usr/bin/git) to link the git alias directory you just created to your real .config/
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'

# 3. Hide untracked files in ~/ from cluttering "git status"
dotfiles config status.showUntrackedFiles no
```
Setup & communicate with this repo via `ssh`, [authenticating with a local private key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux).

``` bash
# 4. Generate SSH key for GitHub auth (if you haven't got one already)
ssh-keygen -t ed25519 -C "you@example.com"
ssh-add ~/.ssh/id_ed25519

# Add the value of ~/.ssh/id_ed25519.pub as an entry in your Github --> Settings --> SSH & GPG Keys, via cat + copy-pasting, or however you'd like.

# 5. Force git on your machine to always push to github with SSH instead of HTTPS
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 6. Set upstream branch as origin main & push via ssh!
dotfiles push --set-upstream origin main
```

then, upon changing my `dotfiles`, can push to github with:
- `dotfiles status`
- `dotfiles add XXXXX`
- `dotfiles commit -m "Update shell and Hyprland config"`
- `dotfiles push` (to remote, via SSH)


## - or... using a dotfiles manager, *comme [`chezmoi`](https://www.chezmoi.io/quick-start/#concepts).*

... which is a tool that essentially creates a copy of your `dotfiles` folder ***outside*** of your `/home` directory (e.g. in `~/.local/share/chezmoi/private_dot_config/`) to act as a place to **stage**, **synchronise** (with `git`) & **manage** changes to your local `dotfiles.`

I think of it as a **remotely-connected playground for your `dotfiles`**, to mess with them, pull them from remote repos etc., **before applying the changes** (via symlinks, copying, or templating) into your **local** home directory (e.g. `~/.config`).

### - To install:
- `sudo pacman -S chezmoi`
- `chezmoi init`
- Check what is & isn't managed by `chezmoi` with `chezmoi managed`/`chezmoi unmanaged`.
- ... then follow steps on [this tutorial](https://www.chezmoi.io/quick-start/#start-using-chezmoi-on-your-current-machine) to connect to your repository & get your first commit. I'm using `chezmoi` to push to the same remote `dotfiles` repo created above, and so just `rebased` my changes (overwriting the old, `chezmoi`-less `dotfiles` from above) to keep it nice and clean.

### - Editing your dotfiles & using `chezmoi`:

**You can edit your `dotfiles` in [multiple ways](https://www.chezmoi.io/user-guide/frequently-asked-questions/usage/#how-do-i-edit-my-dotfiles-with-chezmoi) with `chezmoi`.**

#### **(`RECOMMENDED`)** You can work and make changes within the locally-created `chezmoi` copy of your `dotfiles`, apply them locally, and push them to remote repo once done.

- Navigate to your `chezmoi` dotfiles copy with `chezmoi cd` (you should be able to tell that it's the `chezmoi`-managed copy - e.g. it's called `private_dot_config` for me).

- Then, once you've made changes and are ready to see them/apply them to your **real** `dotfiles` (e.g to see changes live made to your desktop GUI), use `chezmoi status` to list all changed files, `chezmoi diff` to check any changes, and `chezmoi apply` to copy the `chezmoi`-managed files over to your ***local*** `dotfiles.` Now, you should see any changes made **reflected on your live system** (after reloading the given services, if applicable)

- *Then²*, once you're ready to update your remote repo with your changes, go through the usual `git commit` process within the `chezmoi`-managed directory.
- `git status` to see all changed files (within the `chezmoi`-managed copy)
- `git add .` (or whatever files you want to add)
- `git commit -m "cool changes`
- `git push origin main`

***However***, you also have the option of...

### **...making changes to your dotfiles normally** (i.e. not within the `chezmoi`-managed copy of your `dotfiles`)

So, after you're finished a [particularly spicy ricing session](https://i.ytimg.com/vi/GlSa_gh8xaQ/maxresdefault.jpg), you can run:

- `chezmoi status` - to see what's changed between your local `dotfiles` and `chezmoi`'s copy.
- `chezmoi add ~/.config/path/to/file.config` - to add any **locally-changed files** to `chezmoi`'s tracked & `git`-managed copy.
- `chezmoi apply -v` to write these local changes to `chezmoi's` working copy of your `dotfiles`.
- Then switch to the `chezmoi`-managed copy with `cd chezmoi`, and go through the usual `git commit` process to **update your remote repo if desired.**


***`chezmoi`, importantly, allows you to do some of the following cool things:***
- Set up your `dotfiles` on a new machine with a single command: 
  `chezmoi init --apply https://github.com/$GITHUB_USERNAME/dotfiles.git` (public repo - private requires [other methods](https://docs.github.com/en/get-started/git-basics/about-remote-repositories#cloning-with-https-urls))
- Using **[templates](https://www.chezmoi.io/reference/templates/)** to manage `dotfiles` between different machines/distros.
- Encrypting your `dotfiles` using **[secrets from your password manager](https://www.chezmoi.io/user-guide/password-managers/)**

