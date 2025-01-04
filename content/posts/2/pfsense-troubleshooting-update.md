---
title: "pfSense Updating Woes: Crisis Aversion"
date: "2023-06-08"
description: "pfSense troubleshooting: Updating to 23.01 from 22.05"
toc: true
math: false
draft: false
categories:
  - pfsense
tags: 
  - PuTTY
  - netgate
---


![donkey](/posts/2/image.png)

Here is a little guide from a nightmare I encountered whilst trying to perform a maintenance update on a pfSense router... I hope my pain and suffering can help someone else :').

> DO NOT MAKE MY MISTAKE - **CREATE A BACKUP FOR YOUR PFSENSE SETTINGS AND STORE IT LOCALLY BEFORE UPDATING!!!!**.

> pfSense DOES create a backup of settings before updating, but accessing it can be problematic to say the least... (see below)

## Context:

I initiated system reboot and upgrade via the pfSense web UI at 1pm on May 15th. Everything seemed to go well in the web UI, until it restarted and then got stuck reassigning the network interfaces.

![](/posts/2/54985275-a984-494f-923a-e9a771db2005.png)

Looked like the update was successful but got confused when it got to reassigning the interfaces (aka which network interfaces were associated with which network config files previously created) resulting in the boot process being interrupted and not completed, meaning no wifi.

To fix, connected to the pfSense box directly via serial (details at end) & navigated to `/cf/conf/backup/` and did a `ls -lah` to find the last auto-backup xml file and see which network settings were assigned to each interface.

By cross-referencing with the previous backup’s XML data of `<interfaces>`, it was determined that these were the previous assignments:

![](/posts/2/21412848-0228-41eb-b793-7045eea4ee70.png)

> note: for opt3&4, it is ixl0 & ixl1, NOT ix10/ix11 (is an l and NOT a 1)

![](/posts/2/4e5fde35-f32e-4abd-912b-a8c8234755e7.png)

> extract from “config-XXXXXXXXXX.xml”

After reassigning the network interfaces, all seemed to be well and the box booted without any noticeable issues into the new OS.

![](/posts/2/16014c4f-ddbf-480a-8731-980e55020531.png)

From here, I reconnected in powershell by IP via SSH to check it was working, and then accessed the webUI. 🥳

***…I can finally breathe again.***

## Connecting Via Serial Cable ([older tutorial here](https://www.youtube.com/watch?v=M0yyyZojg3M))

1. Get micro-usb to usb cable.

2. Plug micro-usb end into pfsense port labelled `“CONSOLE”`.

3. Plug usb end into laptop usb port. may need to try a few different ones if the steps below don’t work.

4. Navigate [here](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads) & download the CP210x Universal Windows Driver ([direct download link](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads)).

5. Unzip it, right click on `silabser.inf` and click `Install`.

6. Once installed, open device manager (on Windows). There should be a new option called `Silicon Labs CP210x USB to UART Bridge` or similar. The **COM[`X`]** at the end will vary depending on which of your device’s serial ports are already in use.

    ![](/posts/2/dc5ecce7-bbd4-41a1-ba7c-d7a766ddb608.png)

7. [Download](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) & open puTTY, and create a session with the following settings, replacing COM6 with whatever **COM[`X`]** your device listed as being open to communicate with `CP210x` in device manager.

    ![](/posts/2/d47049f6-8c1b-45f7-b74e-5aaa7a3c0bcb.png)

8. Go to `Session` → `Logging` and track all output to a log file saved locally on your machine.

    ![](/posts/2/c526fa04-bf4d-4af5-bf35-3814c46ea260.png)

9. Click `Open` and session should begin.

    ![](/posts/2/43a6dee8-d8a4-42fa-be42-2ffdf14fdac3.png)

## Alternative solutions:

Thankfully a restore point was (and should, by default on pfSense installations) automatically made right before the update, meaning if worst comes to worst, we could:

1. copy the last restore point (config file, found at `/cf/conf/backup/`) to local machine by: turning on the putty output logger, then opening a session & navigating to the config file & `cat`-ting it, saving the output to the local machine. Then:

    a. EITHER completely re-install the 22.05 version of pfSense via USB flash, get network settings reset, then SCP the restore point config file to it and assign it as settings OR used webUI backup option to reload the settings.

    b. OR try and roll back to the previous version (22.05) by assigning the network interfaces randomly and getting to the pfsense options screen, selecting option `15) Restore recent configuration`, and then using the backup config file as the desired settings.

    in case of further panic... here is a link to further [troubleshooting](https://agix.com.au/restore-pfsense-from-backup-using-the-cli-command-line/). I wish you well on your journey, and hope it is substantially shorter than mine was 🙃😭.

---

**DISCLAIMER:** *I would consider this a LEGACY POST of mine, written a long time ago. Please excuse any typos, errors or lapses in memory/judgement - as it was added to the site from the archives, just to put everything in one place. Thankq for your understanding 🙇‍♀️*

---