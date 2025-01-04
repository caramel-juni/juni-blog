---
title: "ESXi, TPMs, fTPMs, & Intel PTT: A love story"
date: "2023-02-15"
description: "Upgrading ESXi Client - plus TPMs, fTPMs, Intel PTT"
toc: true
math: false
draft: false
categories:
  - esxi
tags: 
  - tpm
---

> Here's a quick little rabbit-hole of upgrading ESXi clients, and a crash course on TPMs and their equivalents! 💛

## - TPM/fTPM/Intel PTT Rundown

- **Trusted Platform Module** - *a secure crypto-processor that generates, stores, and limits the use of cryptographic keys required to access system files!!!!!* [(reference)](https://www.onlogic.com/company/io-hub/tpm-for-windows-11-what-is-it-and-what-about-intel-ptt-and-amd-ftpm/).

The traditional **TPM** is a physical security & encryption-focused chip on the motherboard, but can also be built into the firmware of the computer's CPU, which are notably **AMD Firmware TPM (fTPM) and Intel Platform Trust Technology (PTT).**

Both of these have been built to **include TPM 2.0 functionality**, but may not be enabled by default in the machine's BIOS, so be sure to check. It's typically found under the `Security` tab/option in BIOS, which are all different in design, so Google is your bestie when discovering how to access a feature on your specific model 🤪.

## - Upgrading ESXi client:

1. **Enable TSM-SSH via ESXi GUI to enable SSH into your machine**

    Via `Manage` --> `Services` --> `TSM-SSH`, then right click and `Run`.
    
    ![](/posts/1/img1.png)

2. [SSH](https://www.tomshardware.com/how-to/use-ssh-connect-to-remote-computer) into your ESXi box, then check [https://esxi-patches.v-front.de/](https://esxi-patches.v-front.de/) for the latest ESXi patches. Patches are cumulative, so download the latest one (at the top, highlighted).

    ![Alt text](/posts/1/img3.png)

    Copy the text from the pop-up generated after clicking the link, and paste it into the shell of the system via SSH. The code is provided below for convenience (lines separated), but DO NOT just copy the second code block, as it will download the version for `build 8.0b-21203435`, which will change over time.

    - `esxcli network firewall ruleset set -e true -r httpClient`

    - `esxcli software profile update -p [YOUR-VER-HERE] \ -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml`

    Systems with an Intel PTT may throw an error of not having a supported TPM 2.0, and if so, check that Intel PTT is enabled in BIOS before attaching `--no-hardware-warning` to the second code block:

    *Revised Code for Intel PTT (and potentially AMD fTPMs):*

    - `esxcli software profile update -p [YOUR-VER-HERE] \ -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml --no-hardware-warning`

    When running the second code block, the update is downloaded, which can take 5-10minutes depending on your internet connection. Cross your fingers, pray to the networking gods, and just wait 🙏.

    ![](/posts/1/img2.png)

3. Finally, run the final code block to adjust the firewall rules now you're done:

    `esxcli network firewall ruleset set -e false -r httpClient`

    Then reboot to apply changes.

4. Once rebooted, SSH into the box and run `vmware -v` to check the version of ESXi and that the update has been successful.

***💛 Hope this was of some help & best of luck! 💛***

---

**DISCLAIMER:** *I would consider this a LEGACY POST of mine, written a long time ago. Please excuse any typos, errors or lapses in memory/judgement - as it was added to the site from the archives, just to put everything in one place. Thankq for your understanding 🙇‍♀️*

---