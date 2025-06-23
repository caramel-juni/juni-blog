---
title: ""
date: 2025-06-15
description: ""
toc: true
math: true
draft: true
categories: 
tags:
---

- https://github.com/silentz/arch-linux-install-guide
- https://www.youtube.com/watch?v=5DHz23VQJxk
- https://www.youtube.com/watch?v=1J_Z_pzzbMo&t=549s


Recently, my friend had played around with installing arch, and I warned her against choosing *it* as an OS for dual-booting due to how notoriously difficult.... Windows can be when playing with other OS's *rudely* occupying its boot drive... let alone one that's as DIY as Arch (comparatively) for the install, and a rolling release model lending to it being somewhat... *unpredictable*.

So, naturally, upon arriving home, I **decided to do exactly that on my main PC.**

Why? Because, like many people in this industry, I'm a masochist and like pain. Or I'm just crazy. And of course, the rose-tinted allure of "*arch btw*." Gotta try it at least once properly, right? So why not go full insane mode.

Now, the first step:

## Reading the Arch Wiki
Yes, I made the mistake (i jest; *blessing*) of **properly reading the Arch Wiki**. It only took me a dozen tries, and my, was it a ride. I learnt a lot, cried a bit, and **puzzled over the discrepancies between it and the myriad of existing, recent YouTube tutorials/install guides for dual booting Arch with Windows** (all advising to create an additional EFI partition to boot Linux from, when on the same disk).

Now, I was **cross-referencing like crazy**, given i was (crazy enough to be) doing this with my desktop workstation despite *very much* realising this was an extremely risky idea. However, after many back-and-forths, the long and short of it came down to: **Read The Fucking Wiki.** It's the **source of truth.**

**YouTube tutorials only get you *so* far** when dealing with edge cases, and *boy* were there a lot of edge cases. And it's *entirely* worth the extra few minutes of your time, *especially* when attempting to dual boot properly and minimise the risk of a Windows update freaking the fuck out after discovering another `EFI` partition and bricking one (if not both) OS's. *Please* make sure to follow the below recommendations to prevent a *lot* of potential future pain:

## Pre-requisites

1. Do **NOT** create another `EFI` partition to boot `Arch` from when there is an existing one created by Windows. 
Even if only a *chance*, with Microsoft's history, the line: *"An additional EFI system partition should not be created, as it may [prevent Windows from booting](https://support.microsoft.com/en-us/help/2879602/unable-to-boot-if-more-than-one-efi-system-partition-is-present)."* [from the Wiki](https://wiki.archlinux.org/title/Dual_boot_with_Windows) **should very much be heeded.**


2. To *ensure* that your **existing Windows** `EFI` partition **has enough space to store the Arch bootloader and any other future ones that you wish to install**.
   
As Windows typically creates a piddly `100MiB` `EFI` partition by default (only `30MiB` of which is typically used by Windows, it leaves little room for expansion or the potential for other bootloaders. Whist linux-based bootloaders don't take up anywhere *near* as much space as Windows' does... they can get large when including `initramfs`, your kernel and any CPU fixes needed per distro... 
Plus, I wanted to try and do it by the book, especially when running out of space on your `EFI` partition increases the risk of boot loaders misbehaving over time, and in some cases, Windows [nuking your desired Arch bootloader](https://www.reddit.com/r/archlinux/comments/1ca2uyk/why_should_i_or_should_i_not_share_efi_in_a/) upon updates)
   
To do this, if you're unfortunate enough to be stuck with the default `100MiB` `EFI` Windows-created partition, I'd recommend using either [MiniTool Partiton Wizard](https://www.partitionwizard.com/partitionmanager/increase-efi-system-partition-size.html) (as spammy and icky as it comes across - *you can uninstall it right after*) or [GParted](https://pureinfotech.com/resize-partition-windows-10-gparted/#resize_partition_gparted_windows10) (advanced use cases - be careful) to **resize your `EFI` boot partition to something like `4GiB` to be safe.** 
	- Now, full disclosure - this is **not to be done lightly**, and you should **definitely make a backup of the EFI partition before you do this**... and not subsist on crossed fingers and good vibes alone, like I did. (thankfully, I haven't run into any issues... *yet*.) 
	- However, I did so after reading & noting down the `EFI System Partition` (`ESP`'s) `PARTUUID` & `UUID` values - which are typically stored in a `NVRAM` firmware entry and ***thus*** are the main values **used to correctly identify the newly-expanded `EFI` partition**, so if anything *did* go wrong, I could still point Windows to find and boot from it.

3. Ensure to **Disable Fast Startup and Hibernation** modes on Windows via `powercfg /H off` in an elevated Powershell prompt, then power cycle (full shutdown) and check Windows registry keys for this feature to ensure they are off. **See [here](https://wiki.archlinux.org/title/Dual_boot_with_Windows#Disable_Fast_Startup_and_disable_hibernation) as to why.**

Now, once my `EFI` partition was resized, listed as "healthy" in the Windows disk manager, and showing up as `4.1G` in the live Arch install, I proceeded to the fun part - **setting up my Arch system**, guided by both [The Wiki](https://wiki.archlinux.org/title/Dual_boot_with_Windows) (read **the entire dual-booting with Windows article before proceeding**) and this (incomplete) guide [here](according to the guide [here](https://nic96.dev/blog/beginners-guide-dual-booting-arch-linux-windows/).).

---
## The "fun" part - Installing Arch!


### Partitioning drives, creating the filesystem, setting the SWAP
(no need to create an `EFI` partition, as will be using expanded windows one)
#### Some Terminology:
- **Partition:** a logical division of a drive that can be treated as its own section (containing its own filesystem, etc.)
- **Filesystem:** a method and data structure defining how an OS can store, organise and access files on a storage device. Different filesystems have different use cases and limitations (`EXT4`, `NTFS`, `FAT32`).
- **SWAP**: disk space allocated for the filesystem to use as virtual memory if/whenever available physical memory (`RAM`) is exhausted.

List your current drives with `fdisk -l`, and check the output **carefully** for your desired install disk (e.g. `/dev/nvme1n1` for me). I chose to use the `TUI` partitioning tool `cfdisk` to create my Arch filesystem, entering the `TUI` with `cfdisk /dev/nvme1n1`.

I then set my partitions: the `root` (`/`) system, and the `SWAP`. If I were starting again, I would've separated my `root` and `/home` partitions to make distro-hopping, security (isolation) & user-file-level backups a little easier, but oh well.

- `/dev/nvme1n1p6 --> 100G Linux Filesystem` --> to be used as `root` system.
- `/dev/nvme1n1p7 --> 30G Linux SWAP` --> to be used as `SWAP`.

I then **initialised the filesystem type**: choosing `ext4` for my `root` (for general balance of performance, reliability and features; but do consider snapshot-friendly alternatives like `brtfs`). 
- `mkfs.ext4 /dev/nvme1n1p6`

Then, I made and activated the `SWAP` partition:
- `mkswap /dev/nvme1n17`
- `swapon /dev/nvme1n1p7`

Now... for the `EFI` partition!
## [Check whether existing EFI partition is the ESP](https://wiki.archlinux.org/title/EFI_system_partition#Check_for_an_existing_partition)
#### Some Terminology:
- **EFI:** 
- **ESP:** 

`fdisk -l`

--> e.g. for me, `/dev/nvme1n13` has TYPE: `EFI System`
mount it to local FS to check that it check it contains `EFI` directory
mount `/dev/nvme1n13` `/mnt`
`ls /mnt`
if it does, good to go! if not see the link above.

### mount root partition to `/mnt`

mount `/dev/nvme1n1p6` `/mnt`

### mount (existing) windows EFI partition (for me, `/dev/nvme1n1p3`) to `/mnt/efi`
`mkdir /mnt/efi`
`mount /dev/nvme1n1p3 /mnt/efi`


- install required packages:
`pacstrap -K /mnt base base-devel linux linux-firmware linux-headers amd-ucode sudo git vim htop curl wget bluez bluez-utils networkmanager gcc fastfetch pipewire make cmake cargo mpv efibootmgr mtools man-db man-pages tealdeer dhcpcd resolvconf` ...

(using providers `pipewire-jack`, `openresolv`, `rustup` when prompted)

- generate [fstab](https://wiki.archlinux.org/title/Fstab) 
`genfstab -U /mnt >> /mnt/etc/fstab`
check with
`cat /mnt/etc/fstab

switch into new arch system with

`arch-chroot /mnt`
*This allows you to run commands as if you had booted into that system - even though you’re still technically running under the host OS (on the live boot drive)*

use `fastfetch` and voila!

## MAKE SURE YOU RUN:
 `mkinitcpio -P` - tries to include all necessary **kernel modules** and **firmware** for the devices your system might need to boot. The warning:

#### change root pw, add (wheel/sudo privs) users, timezone, locale, hostname, network on startup
- set root password with `passwd`
- `useradd -m -g users -G wheel,storage,power,audio,video -s /bin/bash juniarch**
- change locale:
- set timezone & synchronise clock
- set hostname - `echo "arch" >> /etc/hostname` and then edit `/etc/hosts` to include localhost as your desired `hostname.local`, e.g.:
	- `127.0.0.1      arch.localdomain     arch`
- `systemctl enable bluetooth` 
- `systemctl enable NetworkManager` --> starts network on boot
- `systemctl enable dhcpcd` (`dhcpcd` will call `resolvconf` (i.e. `openresolv`) as needed.)

### install & enable bootloader
as am booting off an existing EPS (windows-created), & whilst it *can* be used with `grub`, apparently `rEFind` is good for a simplified UI & autodetecting existing EFI boot entries (on `UEFI`), so I thought I'd try that.

`pacman -S refine`
run setup script --> `refind-install`. Should set it as default boot manager. check with `efibootmgr` - `BootOrder:` line.

This also runs `mkrlconf` as `root`, and will attempt to find your kernel in `/boot` and automatically generate `refind_linux.conf` accordingly.

## HOWEVER!!!
- ⚠ **Do not trust the auto-generated `/boot/refind_linux.conf`** ⚠
	- See why this is the case [here](https://www.reddit.com/r/archlinux/comments/1bwbb6p/refind_not_configuring_root_partition_right/) -- as the script detects `ISO` boot conditions instead of the **underlying file system.** ***Is a known flaw!***
	  See video [reconfiguring it properly here](https://www.youtube.com/watch?v=KW1jbeLdzB8 ).
Check and fix it manually by filling in your corresponding new `/root` system's Partition UUID (`PARTUUID`):
- Get correct PARTUUIDs & UUIDs of root disk with `blkid /dev/nvme1n1p3` (for example).
- Generally is recommended to use `PARTUUID` over `UUID` as (apparently):
	1. `PARTUUID` doesn't depend on the presence of a filesystem (as is assigned by `GPT`), and will thus **exist even on empty partitions**.
	2. It tends to be **more stable for dual-booting** as it will **always point to the exact partition** instead of a filesystem (which may move around... looking at you, Windows Updates).
	3. Can prevent `kernel panik !!1!` when booting as `PARTUUID` is sometimes resolved **earlier** than filesystem `UUIDs`.
    example within `/boot/refind_linux.conf`:
``` bash
"Boot Arch Linux" "rw root=PARTUUID=87463764-aaaa-aaaa-aaaa-76234876486723 add_efi_memmap initrd=/initramfs-linux.img"`
```

**You only need to do this once**, then it will work on reboot and rEFInd will autodetect your Arch kernel.

Then, feel free to edit any further settings you'd like to change (default boot order, icons, etc.) in `/efi/EFI/refind/refind.conf


#### One extra step: Check that you didn't overwrite or delete any existing EFI entries (like`/EFI/Microsoft` - that would be bad)

- Mount your underlying `OS`'s `EFI` partition and check that (for example) the Windows bootloader files are still there:
```bash
sudo mount /dev/sdXY /mnt  # sdXY or nvmenX = your EFI partition
ls /mnt/EFI/Microsoft/Boot/
# Should show files like bootmgfw.efi
sudo umount /mnt

```

Now - to reboot into your new Arch system!

- Exit `chroot` environment with `exit`
- Unmount all partitions: `umount -lR /mnt
- Now the directories you previously mounted should no longer be listed as mountpoints when running `lsblk`.
- Finally, `shutdown now` & **eject pendrive**

### What happened next...
And of course, as the `rEFind` boot menu loaded up (had to manually change boot order in `UEFI BIOS` as it overwrote that described by `efibootmgr`) and i selected the glorious Arch logo, I got dropped into an emergency shell with the error:
`device "PARTUUID" not found`
... despite that device `PARTUUID` listed being the correct one for my `/root` drive.

### Time to troubleshoot:
After booting into Arch live ISO, mounting disks as specified before (existing `EFI`, `/root`, and setting `SWAP` on), and then using `arch-chroot /mnt` to enter the live preview system, I tried...
- running `mkinitcpio -P` (as boot drive is `nvme` & needs additional auto-setup)
- uncommenting the `extra_kernel_version_strings` field in`[esp-directory]/EFI/refind/refind.conf` & adding those seen in the example on the [Arch Wiki (rEFind)](https://wiki.archlinux.org/title/REFInd#For_kernels_automatically_detected_by_rEFInd), 
	- Because, *"For `rEFInd` to support the naming scheme of Arch Linux [kernels](https://wiki.archlinux.org/title/Kernels "Kernels") and thus allow matching them with their respective initramfs images, you must uncomment and edit `extra_kernel_version_strings` option in `efi/EFI/refind/refind.conf`"* 
... but still to no avail. 

But I had not yet tried my greatest strategy.

I **made a troubleshooting plan**, and then **took a break** (when it hit 2am).
And then **came back the next day to look over it.**

Surprise surprise, the answer lay within... 

#### THE PROBLEM: FAILING TO SPECIFY THAT I WAS USING `PARTUUID` IN `/boot/refind.conf`...
Somewhat obviously (yet not to my 2am brain), you need to DECLARE the kind of identifier you're booting from. i.e., include `PARTUUID` within the line `root=PARTUUID=87463764...`
 e.g. (fixed version)
 ``` bash
"Boot with standard options" "rw root=PARTUUID=87463764-aaaa-aaaa-aaaa-76234876486723 add_efi_memmap"
 ```

So, um. Yeah. Fun.

I then cleaned up & removed old boot images from my `/efi/EFI/` directory (an old, now wiped Manjaro install), and tinkered around with the `[esp-directory]/EFI/refind/refind.conf` file to change my boot options (see [here](https://rodsbooks.com/refind/configfile.html) for a list of all parameters & how to use them).

### And then, bam! 
- **Arch Linux** installed
- ...Dual booting from the **same drive** with an **existing Windows 11 installation**
- ...using the **same EFI partition** as Windows
- ...using a bootloader (`rEFind`) capable of [launching kernels as](https://wiki.archlinux.org/title/REFInd) `EFI boot stubs` ("graceful", some might say) 
- ... that was made *way* harder than it needed to be by the absence of one freaking word (`PARTUUID`) 
All without bricking my system.

***Oh, isn't tech just oodles of fun?***




---

# Linux Ricing Plan:
- [Getting started with dotfiles](https://webpro.nl/articles/getting-started-with-dotfiles)
- [Dotfiles management resources & tutorials](https://github.com/webpro/awesome-dotfiles?tab=readme-ov-file)
- [Linux Ricing Guide](https://www.youtube.com/watch?v=jFz5gLqv-FM) (denkii)

### Components:
- dotfile manager
	- [chezmoi](https://www.chezmoi.io/what-does-chezmoi-do/)
	- [yadm](https://github.com/yadm-dev/yadm)
- login manager
	- `ly`
- window manager
	- `wayland`
	- `i3`
- compositor
	- `picom` (transparency, shadows,)
- notification daemon
- terminal emulator
	- `kitty` - renders images, very fast, multiple tabs, sessions
- screenshot tool
	- `flameshot` (simple but a lot of capabilities)
- file browsers (`CLI`)
	- `nnn` - fast, lightweight `CLI` file browser
- file browser (`GUI`)
	- `Thunar` (from xfce)
- blue light filter
	- `redshift`
- web browser
	- `firefox`
	- `librewolf`
- code editor
	- `nvim`
	- `lazyvim`
	- `nvchad`
	- `vscode`
- fonts
	- `Hack` (terminal)
	- `Nerd Fonts`
- wallpapers
	- `r/unixporn`
	- `arch discord`
- GTK (gnome software) theme manager
	- `lxappearance`
- system maintenance script
	- https://www.youtube.com/watch?v=o03_cfOnl84
	- see [dotfiles here](https://github.com/kurealnum/dotfiles)


## Ricing steps taken:

**snapshot tool: timeshift**
- `yay -S timeshift`
- `sudo timeshift --list-devices`
	  This tells you which partition Timeshift sees as eligible (e.g., `/dev/sda2`).
- `sudo timeshift --check`
- modified config to include `/home` directory in backup, and only exclude the cache: `**/home/.cache`
- `sudo timeshift --create --comments "Full system + home pre-rice" --tags D`
- create a small file like `hello.txt` in home directory, then restore previous snapshot with: `sudo timeshift --restore`. make sure **NOT** to reinstall grub2 bootloader, as not using grub.




---

# NEW ARTICLE
## Tracking & syncing dotfiles

### Using Git + Github, & tracking dotfiles with an alias
- https://wiki.archlinux.org/title/Dotfiles#Tracking_dotfiles_directly_with_Git

``` bash
# 1. Create a bare Git repo to track dotfiles
git init --bare ~/.dotfiles

# 2. Create an alias to simplify dotfiles management
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'

# 3. Hide untracked files from cluttering status
dotfiles config status.showUntrackedFiles no

# 4. Generate SSH key for GitHub auth (one-time)
ssh-keygen -t ed25519 -C "you@example.com"
ssh-add ~/.ssh/id_ed25519
# Add ~/.ssh/id_ed25519.pub to GitHub under SSH keys

# 5. Force GitHub to always use SSH instead of HTTPS
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 6. Set upstream branch as origin main
dotfiles push --set-upstream origin main
```

then, upon changing my `dotfiles`, can push to github with:
- `dotfiles status`
- `dotfiles add XXXXX`
- `dotfiles commit -m "Update shell and Hyprland config"`
- `dotfiles push` (to remote, via SSH)


### or... using `chezmoi`
- https://www.chezmoi.io/quick-start/#concepts

... which essentially creates a copy of your `dotfiles` folder ***outside*** of your `/home` directory (e.g. in `~/.local/share/chezmoi/private_dot_config/`) to act as a place to **stage**, **synchronise** (with `git`) & **manage** changes to your local `dotfiles.`

I think of it as a **remotely-connected playground for your `dotfiles`**, to mess with them, pull them from remote repos etc., **before applying the changes locally** (via symlinks, copying, or templating) into your **local** home directory (e.g. `~/.config`).

**To install:**
- `sudo pacman -S chezmoi`
- `chezmoi init`
- ... then follow steps on [this tutorial](https://www.chezmoi.io/quick-start/#start-using-chezmoi-on-your-current-machine) to connect to your repository & get your first commit. i've linked it to the same `.dotfiles` repo, and just `rebased` my changes (overwriting old, `chezmoi`-less repo created above) to keep it clean.

**You can edit your `dotfiles` in [multiple ways](https://www.chezmoi.io/user-guide/frequently-asked-questions/usage/#how-do-i-edit-my-dotfiles-with-chezmoi) with `chezmoi`.**

**(`RECOMMENDED`)** You can work and make changes within the locally-created `chezmoi` copy of your `dotfiles`, after jumping to it with `chezmoi cd` (you should be able to tell that it's the `chezmoi`-managed copy - e.g. it's called `private_dot_config` for me).

Then, once you've made changes and are ready to see them/apply them to your **real** `dotfiles` (e.g to see changes live made to your desktop GUI), use `chezmoi status` to list all changed files, `chezmoi diff` to check the changes, and `chezmoi apply` to copy the `chezmoi`-managed files over to your ***local*** `dotfiles.` Now, you should see any changes made reflected on your live system (after reloading the given services, if applicable)

*Then²*, once you're ready to push them to your remote repo, go through the usual `git commit` process within the `chezmoi`-managed directory.
- `git status` to see all changed (`chezmoi`-managed) files 
- `git add .` (or whatever files you want to add)
- `git commit -m "cool changes`
- `git push origin main`

***However***, you can also just **make changes to your dotfiles normally** (i.e. not within the `chezmoi`-managed copy of your `dotfiles`) - then once done, running:

- `chezmoi status` - to see what's changed between your local `dotfiles` and `chezmoi`'s copy.
- `chezmoi add ~/.config/path/to/file.config` - to add any locally-changed files to `chezmoi`'s tracked & `git`-managed copy.
- `chezmoi apply -v` to write these local changes to `chezmoi's` working copy of your `dotfiles`.
- Then switch to the `chezmoi` copy with `cd chezmoi` and go through the usual `git commit` process to update your remote repo if desired.


**Can do more cool stuff like:**
- Set up a new machine with a single command: 
  `chezmoi init --apply https://github.com/$GITHUB_USERNAME/dotfiles.git` (public repo - private requires [other methods](https://docs.github.com/en/get-started/git-basics/about-remote-repositories#cloning-with-https-urls))
- Use **[templates](https://www.chezmoi.io/reference/templates/)** to manage files between different machines/distros
- Encrypting your `dotfiles` using **[secrets from your password manager](https://www.chezmoi.io/user-guide/password-managers/)**
- Check what is & isn't managed by `chezmoi` with `chezmoi managed`/`chezmoi unmanaged`.




## further steps to troubleshoot:
- [x] [Arch Wiki rEFind](https://wiki.archlinux.org/title/REFInd#For_kernels_automatically_detected_by_rEFInd) - *"For `rEFInd` to support the naming scheme of Arch Linux [kernels](https://wiki.archlinux.org/title/Kernels "Kernels") and thus allow matching them with their respective initramfs images, you must uncomment and edit `extra_kernel_version_strings` option in `efi/EFI/refind/refind.conf`"* 
  ``` bash
extra_kernel_version_strings "linux-hardened,linux-rt-lts,linux-zen,linux-lts,linux-rt,linux"
	```
	- ***Warning:** Without `extra_kernel_version_strings` set, rEFInd will incorrectly pass the first initramfs it finds as the `initrd=` kernel parameter, instead of using the correct initramfs for the kernel. This will result in a failure to boot since the matching loadable kernel modules will not be available.*
	- *Without the above `extra_kernel_version_strings` line, the `%v` variable in `refind_linux.conf` will not work for Arch Linux [kernels](https://wiki.archlinux.org/title/Kernels "Kernels").*



- try `UUID` instead of `PARTUUID`, as seen below:
	- https://www.youtube.com/watch?v=KW1jbeLdzB8&t=29s
	- https://www.rodsbooks.com/refind/linux.html#efistub
- [x] run `efibootmgr -v` to check EFI partition layout
- check - [You must place your kernels in a directory other than the one that holds the main rEFInd .efi file. This is because rEFInd does not scan its own directory for boot loaders.](https://www.rodsbooks.com/refind/linux.html#efistub)
- check if using encryption or LVM
--> ==try and install grub instead
- https://www.youtube.com/watch?v=tCGL_FY3xeM&t=1750s