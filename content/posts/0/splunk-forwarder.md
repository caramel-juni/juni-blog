---
title: "Splunk Forwarders with Multiple Indexes"
date: "2023-01-23"
description: "Setting up a single Splunk Forwarder to send different data to multiple indexes"
toc: true
math: false
draft: false
categories:
  - splunk
tags: 
  - logging
---

## - Pre-requisites:

1. Have a working Splunk instance (Splunk Enterprise, in my case) to connect to. There are plenty of tuts for this online.

2. Have installed a universal forwarder on the endpoint that you want to monitor (see here, an excellent post which will get you most of the way through setting up Splunk to analyse Suricata & pfSense logs)

> Note: There are some steps specific to my use case, which are marked accordingly with ***[OPTIONAL]***. These can be ignored if you're just trying to configure a universal forwarder to send data to multiple indexes in Splunk.

## - Detailed Steps:

1. Create the desired index in Splunk (Settings --> Indexes). I named mine `ids_lan` as I am using an Intrusion Detection System (IDS) to monitor my LAN network on pfSense. You can leave all the index settings as default for now.

    ![Indexes](https://cdn.hashnode.com/res/hashnode/image/upload/v1674202112483/cce019ab-74e4-4018-b49c-e329f0b52e60.png)

    ![index name ids_lan](https://cdn.hashnode.com/res/hashnode/image/upload/v1674202139846/a4d02e8f-7c8b-4155-9d19-571085ee88b4.png)

2. ***[OPTIONAL]*** Go to pfsense web UI, and create & configure the instance that you want to monitor. Once up and running, go to 'Logs View' and select the instance to view.  
	![](/posts/0/splunk3.png)

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674202427414/c05ed6d0-aba7-4be3-bd8b-c790429611cc.png?auto=compress,format&format=webp)

    Note the file path and importantly, the folder name where logs are sent for that instance. My path is `/var/log/suricata/suricata_em125470/eve.json` and the folder name is `suricata_em125470`.


3. SSH into your VM/machine with the splunk forwarder installed and modify the inputs.conf file. In my case it's found in `/opt/splunkforwarder/etc/apps/TA-Suricata/default`, but that's because I'm using the [**TA-Suricata**](https://splunkbase.splunk.com/app/2760) app to make my Suricata logs Splunk-readable (matching Splunk's Common Information Model [CIM]).

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674202463423/f0338623-9179-4154-afab-6eff2dfe8af1.png?auto=compress,format&format=webp)

    In most cases, you should navigate to `/opt/splunkforwarder/etc/system/local` and create an inputs.conf file if there isn't already one. This overrides all of the defaults located in `/opt/splunkforwarder/etc/system/default`, and it's [**best practice**](https://docs.splunk.com/Documentation/Splunk/9.0.3/Data/Monitorfilesanddirectorieswithinputs.conf#:~:text=The%20inputs.,a%20stanza%20to%20the%20inputs.) to modify files within `/local` instead when you're not using an extra app like I am.

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674202812915/47e63a91-6b6b-48f3-9a94-f163b7de5b19.png?auto=compress,format&format=webp)


4. Within the `inputs.conf` file, create entries like so (as many as needed), making sure to reference the new folder that was created in step 3 (`suricata_em125470`).

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674203175586/f3a9904a-2e33-4b61-b964-fd9207851993.png?auto=compress,format&format=webp)

    Additionally, set `index = ids_lan` , or whatever index you created in Step 1.

~

> **For a more general installation, here is some sample code (excluding suricata-specific options).**


## - Code Dump:
---

`[monitor://path_to_your_monitored_file_here]`

`disabled = false`

`index = your_index_from_step_1`

`host = your_splunk_instance_name`

---

***(Your splunk instance name is found by going to `Settings --> Server Settings --> General Settings` in the splunk web UI)***

1. Navigate to the `/opt/splunkforwarder/bin/` directory and run the command `./splunk restart` to restart the splunk forwarder. **A reboot of the system is (likely) not sufficient.**

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674203970829/b7c520ae-14df-4fdb-910c-2668fc8ac35f.png?auto=compress,format&format=webp)

2. Upon restarting, within the same directory (`/opt/splunkforwarder/bin/`) check that Splunk is running with `./splunk status`. If it isn't, run `./splunk start`. Verify there are logs in the folder you just linked to (in my case, `suricata_em125470`), which can be done either by navigating to the path where they are stored, or in my use case, via `Log Contents` in the pfSense web UI (see step 3).

3. Wait a few minutes or so, and then run a splunk search for a timeframe for which you know there are log entries for, with something like `index="your_index_from_step_1"` . Check the time stamp matches when you expect the last log to have been from, and voila!

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674204285133/70586cb2-a6d8-4ae9-8273-01f5add4dc02.png?auto=compress,format&format=webp)



💛 ***Best of luck, I hope this helps some people and avoids too much keyboard-bashing!*** 💛

---

**DISCLAIMER:** *I would consider this a LEGACY POST of mine, written a long time ago. Please excuse any typos, errors or lapses in memory/judgement - as it was added to the site from the archives, just to put everything in one place. Thankq for your understanding 🙇‍♀️*

---