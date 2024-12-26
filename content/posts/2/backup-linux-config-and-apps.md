---
title: "Backing up linux .config & apps to move to a new device/distro"
date: "2024-12-25"
description: "a distro hopper's delight"
toc: true
math: false
draft: false
categories:
  - linux
tags: 
  - distro hopping
  - backup
  - rsync
---

# burn it all down... or?

Ahh, a tale born from the first time that I dipped my toes into the weird, wide and wonderful world of distro-hopping. Because sometimes, instead of building it all from scratch again (like so many of us are fond of doing), bringing your old config, notes of a previous home, with you is desirable. Because don't lie - we won't get those hours spent tweaking shell configs to look *just* how we like it back.

In any case - the following (somewhat high-level) overview should get you up and running on a new system/distro fairly quickly, in an environment 

Basically, most user settings (from my research - some may be hidden in other corners, but this got me back to a similar place) are stored in `/home/[user-name]/.config/.

So, for me, this was at `/home/juni/.config/`. So, simply copy that folder to an external drive or over the network, and paste it in the corresponding place on your new system.

## - Copying over .config

1. `cd /home/[user]/`
	- navigate to the user's directory where the `.config` folder is stored.

2. `sudo tar cvzf configs-backup.tgz .config/
	- creates a compressed archive (`configs-backup.tgz`) of the `.config` folder with `tar`, and passing `cvzf` as parameters: 
		- `c` - `c`reate a new archive
		- `v` - enable `v`erbose output, to monitor the progress
		- `z` - compress with the g`z`ip algorithm
		- `f` - specifies the name of the created archive `f`ile (in this case, `configs-backup.tgz`)
		Alternatively, you could use a tool like `rsync` to copy the **entire** `/home/` folder to an external ssd, although this can take a **long time** depending on its size. I'd recommend `rsync` over just copying with `cp`, as `rsync` copies all files **whilst** retaining `owner/group/other` file permissions. 

3. If connecting an external SSD to copy to:
	`sudo fdisk -l
	- lists the connected disk drives and their corresponding filesystem location - like `/dev/sda1)

4. `sudo mkdir -p /mnt/externalssd
	- creates a folder on your computer's filesystem to act as a **mount point:** i.e. a place where you can access files stored on a mounted external SSD.

5. `sudo mount -t exfat /dev/sda1 /mnt/externalssd
	- Mounting the SSD (the device we found at `/dev/sda1`) 'in' this new folder created in the previous step, allowing all the files on it to appear in `/mnt/externalssd`.

6. You should now be able to navigate there with `cd /mnt/externalssd` and run a `ls` to show the SSD's existing contents. Then, copy the compressed .config file with `cp /home/[user]/configs-backup.tgz /mnt/externalssd` (may require prepending `sudo` depending on user permissions) - and you're done!.



If you opted for `rsync` instead above:

`sudo rsync -avh --progress /home/[user]/ /mnt/externalssd/home-backup`
	- `a` - preserves file `a`ttributes & ensures a **mirror copy** is created, including permissions, symlinks, etc.
	- `v` - enable `v`erbose output, to monitor the progress
	- `h` - ensures output is `h`uman-readable
	- `--progress` - displays real-time progress for troubleshooting purposes.

7. Now just unmount the drive with `sudo umount /mnt/externalssd` (or don't - live on the edge ;), plug it into new machine/distro, and copy the file you created over into `/home/[new-user]/` with `cp`. 
   
   Make sure to de-compress the file (if you used `tar`) with `tar xvzf configs-backup.tgz`, so it can be read by the system! 
   
   Then reboot, and your settings should be re-applied! :3

## - BONUS: Grabbing a list of installed packages to re-install

Optionally, if you want to grab a list of all packages/apps installed on your current distro to bring over and auto-install on your new one, run the following:

**Debian-based distros (e.g. Ubuntu, Kali, etc.):**
1. `dpkg --get-selections > installed-packages.txt`
	- saves a list of all packages to `installed-packages.txt`. Save this on an external SSD or transfer to the new machine via the network.
**On the New Machine/Distro:**
2. `sudo apt update`
3. Navigate to where `installed-packages.txt` is stored (on the local machine), and run `sudo dpkg --set-selections < installed-packages.txt`
4. Run `sudo apt-get dselect-upgrade`

The process is similar for distributions using different package managers like `yum`, `pacman`, or `rpm`, the concept is the same but the commands will differ slightly. A little net/manual searching will fix you up :P.

## - Related helpful articles:
- https://help.ubuntu.com/community/BackupYourSystem/TAR
- https://askubuntu.com/questions/7809/how-to-back-up-my-entire-system

---

**DISCLAIMER:** *I would consider this a LEGACY POST of mine, written a long time ago. Please excuse any typos, errors or lapses in memory/judgement - as it was added to the site from the archives, just to put everything in one place. Thankq for your understanding 🙇‍♀️*

---
