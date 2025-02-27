---
title: heading towards the High Seas
date: 2025-01-27
description: a documented descent into the state of torrenting in 2025
toc: true
math: true
draft: true
categories: 
tags:
---
## Why Torrenting?


## Preparing the NAS




## Configuring TrueNAS Scale:
- [Getting Hardlinks to work on TrueNAS Scale](https://www.youtube.com/watch?v=fAdGchuUsWY) - aka, creating one **DATASET** per pool and then creating **folders & subfolders** underneath them via the TrueNAS Scale CLI.
- Access CLI:
  ![](posts/14/Screenshot%202025-01-27%20at%209.30.18%20pm.png)
  ![](posts/14/Screenshot%202025-01-27%20at%209.33.51%20pm.png)
- Create Folders:
  ![](posts/14/Screenshot%202025-01-27%20at%209.50.01%20pm.png)
- Change perms for folders:
  ![](posts/14/Screenshot%202025-01-27%20at%209.49.07%20pm.png)
  ![](posts/14/Screenshot%202025-01-27%20at%209.48.23%20pm%201.png)


1. create main pool `tank` containing all 3 HDDs in raidz1 array. `tank` also acts as a dataset (aka filesystem).
2. DO NOT NEST DATASETS if you want to **utilise hardlinking between them** (see above). Instead, create **folders** via the TrueNAS CLI.
		e.g., with the root dataset `tank`, can allow torrents to download files into `tank/torrents/movies`, and then once done, **still continue to seed the file from there** whilst **creating a hardlink to it in** `tank/media/movies` to be accessed over the network share. this way, the file data does not have to be copied over into **a different dataset**. 
		
	To achieve this, go to the TrueNAS shell & create folders/subfolders, and change permissions on them so the "apps" group can access everything in the `media` folder (to stream) and your bittorrent client can access everything in the `torrents` folder. change according to setup. may need to go to `users` & enable `sudo` commands for the `truenas_admin` user to do this.

### Users & Groups
Now this will be incredibly tailored to the individual, but here's how I set things up:
**Users:**
- `root` user (created by default, pw disabled)
- `truenas_admin` (created by default, for sudo commands)
- `juni-nas` (my admin account for GUI access, can't run sudo commands)
- An account per family member who might access the SMB shares (ensuring all of them are **SMB-enabled accounts** via the checkbox when creating them). For my use case, I used their name as the local user & password credentials (for ease of use given it's all on my LAN, not recommended in critical/high security systems... obviously). No additional permissions.
![](posts/14/Screenshot%202025-01-28%20at%207.51.19%20pm.png)
#### Groups:
- `truenas_admin` (created by default, contains that user)
- `juni-nas` (created by default, contains that user)
- `X-fam` (group containing all family members, Samba/SMB authorised)
![](posts/14/Screenshot%202025-01-28%20at%208.04.23%20pm.png)

This way, I can set up ACLs to the SMB shares as follows: with the `X-fam` group containing all of my non-privileged family users having access to the shared folder (NOT entire dataset) at `/mnt/tank/media`. 

![](posts/14/Screenshot%202025-01-28%20at%208.06.04%20pm.png)
![](posts/14/Screenshot%202025-01-28%20at%208.06.28%20pm.png)
The filesystem permissions for the `/mnt/tank/media` folder (important for reading/writing to it) are as follows: 
- it's **owned** by the `apps` user (& its group), so that applications like the 'ARRs' can freely write/read to it.
	- TrueNAS OPEN_POSIX ACL requires, rather messily, the following ACLs to be set (corresponding in my case to the folder's owner, `apps`): the `User Obj`, `Group Obj`, `Other`, `User Obj - default`, `Group Obj - default`, `Other - default` and a `Mask - default`. Explaining why is [beyond the scope](https://www.truenas.com/community/threads/scale-beta-acl-and-permissions-errors.93884/) of this tutorial/me at the moment, haha ~.
- importantly, I set the `X-fam` group (containing my family users) to have read, write & execute access - allowing them to **modify/add/delete files on the share at a filesystem level**
![](posts/14/Screenshot%202025-01-28%20at%208.10.14%20pm.png)

After all of this, I was able to access the SMB share via windows File Explorer (after turning the network sharing on, when prompted), by logging in with any of my given family member's credentials!

#### Creating SMB Share
Guide in [Docs](https://www.truenas.com/docs/scale/24.04/scaletutorials/shares/smb/#how-do-i-add-an-smb-share):
- create new SMB account in Users - **as anyone on local network will require credentials to access the share locally**. Can be something simple (pending your security posture) - like user: `media` pass: `media` (as whoever is accessing is already on your internal network)
- go to "dataset"
- select existing one (or create new one & set share type to SMB share)
- for [smb shares](https://www.truenas.com/docs/scale/24.04/scaletutorials/shares/smb/): watch https://www.youtube.com/watch?v=59NGNZ0kO04&t=4s
- for apps: watch 

### NFS Share
Guide in [Docs](https://www.truenas.com/docs/scale/24.04/scaletutorials/shares/addingnfsshares/#adding-nfs-share-networks-and-hosts):
- need to figure out permissions - as any registered user on the TrueNAS system seems to be able to access via NFS, and cannot control ACLs as granularly as with SMB when using the default SYS level authentication (aka using **locally acquired UIDs and GIDs** - no cryptographic security.)
	- Seems to require something like a KDC & kerberos implementation to properly enforce access control for users, see [here](https://www.truenas.com/community/threads/mapall-root-safe.110022/)
	- can potentially limit to [a specific network/fixed IPs/hostnames](https://www.truenas.com/community/threads/map-nfs-clients-to-servers-owner-group.45319/)


### Snapshots
Pretty simple - setting up differential (so only stores any *changes* in files) ZFS snapshots for each dataset. Checked the `recursive` option, and unchecked `save empty snapshots` as only want those with changes/substance.
![](posts/14/Screenshot%202025-01-28%20at%2011.15.40%20pm.png)
- [ ] *in future - i'll want to sync these snapshots somewhere else (e.g via rsync to another computer/my PC)*

### Set up alerting:
1. add system email account to send emails from, via System --> General Settings page, at the bottom. Can use gmail OAuth or own/third party SMTP server. Email alerts will **come from this address**.
2. go to settings --> alert settings and add the email to send **to** when a specific level of alert (warning & up, etc.) is triggered.

### S.M.A.R.T disk health checks
Set them to periodically run for all disks, every sunday at 4am, ensure they are not scheduled at the same time or day as any resilver/scrub operations, as that can **take disks offline**. Can read more about why you should do these, and the types, [here](https://www.truenas.com/docs/core/13.0/coretutorials/tasks/runningsmarttests/).
![](posts/14/Screenshot%202025-01-28%20at%2011.44.45%20pm.png)
![](posts/14/Screenshot%202025-01-28%20at%2011.44.31%20pm.png)


## Tuning your TrueNAS
### ARC - Adaptive Replacement Cache
- RAM available & used to store & access files (determined via an algorithm, most often used, etc.) instead of having to retrieve them from disc(s). Greatly improves file read/write speeds. **A loose rule: 1GB RAM for ~ 1TB DISK** 
  ![](posts/14/Screenshot%202025-01-28%20at%207.44.23%20pm.png)
  Linux *used to* only reserve 50% of RAM by default to use for ARC (could be fixed with a command, was just how linux worked with ZFS) - but as of TrueNAS 24.04 release, this is **no longer an issue**. 
### LARC2 Cache Drive
Level 2 Cache Drive - will act as extra ARC for your main pool.
- I used a 2TB nvme m.2 SSD as a striped cache drive in my main HDD pool, to improve read speeds. if this fails, it doesn't take the pool down with it, as it's only caching copies of stored files and provides no "extra" storage space - which is a relief.
- from my understanding, an SLOG drive *can* improve synchronous write speeds but that will likely not be of massive use to me, given my network speeds (100mb/s down & 20mb/s up on a **very good day**) will bottleneck the HDD write speeds (150mb/s). plus, i'd likely want to mirror them for redundancy as data being written to the larger pool *can* be lost if only on the SLOG drive.


### The 'Arrs'!
- **Radarr (movies)**
- **Sonarr (tv)**
- **Lidarr (music)**
- **Readarr (books/audiobooks)**

All the above configs have additional storage set as follows (for the respective subfolder - movies, tv, books, music - depending on the application's purpose):
![](posts/14/Screenshot%202025-01-29%20at%2010.07.33%20am.png)

- **Prowlarr (indexer from torrents)** 
No additional storage configuration needed (as will not read/write files, just indexes them to send to other apps)


- **Homarr**
Generate encryption key to use with it:
![](posts/14/Screenshot%202025-01-29%20at%2010.35.37%20am.png)
![](posts/14/Screenshot%202025-01-29%20at%2010.36.44%20am.png)



### Recyclarr:
- [Video Guide](https://www.youtube.com/watch?v=Zsd65S0G2P0)
- [Text Guide](https://wiki.serversatho.me/en/Recyclarr)


## To Do:
- [x] organise & standardise naming conventions in qbit, radarr & sonarr
- [x] test our sonarr & downloading there
- [x] video quality settings settle on
- [x] sort out socks5 proxy
- [ ] set up tdarr (save space & transcode)
- [x] create a search feature to auto-grab NEW torrents on TL (new ones) that have between 1-30 seeders and 20+ leechers --> use [autobrr](https://wiki.torrentleech.org/doku.php/autobrr?s[]=rss)
- [x] [hardening truenas scale](https://www.youtube.com/watch?v=u0btB6IkkEk&list=TLPQMzEwMTIwMjWkITj8P9yTow&index=3)
- [x] jellyfin & jellyseer remote access w cloudflare tunnel (ask joel about their plex setup & media quality)
- [x] map to torrentleech & start freeseeding
- [ ] set up the rest of arrs (lidarr, readarr etc.)
- [x] test hardlinks working
- [ ] uptimekuma finish configuring & explore
- [ ] pihole fully configure
- [x] dashboard to jump between these
- [ ] rsync setup for backups
- [ ] ~~fully setup jellyfin - hardware transcoding, etc.~~
- [x] import older movies into radarr
- [x] limit upload speeds during the day/peak hours
- [ ] configure [netdata stats dashboard](http://192.168.0.25:20489/spaces/juniper-space/rooms/all-nodes/dashboards/juninas#metrics_correlation=false&after=-900&before=0&utc=Australia%2FAdelaide&offset=%2B10.5&timezoneName=Adelaide&modal=addChartModal&modalTab=&modalParams=&selectedIntegrationCategory=deploy.operating-systems&force_play=false&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1%5Bnetdata_agent_local%5D--chartName-val=menu_System-0&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1%5Bnetdata_agent_local%5D-nodesView-nodeIdToGo-val=menu_Live&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1%5Bnetdata_agent_local%5D--selectedNodeIds-arr=&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1-nodesView-nodeIdToGo-val=menu_Live&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1-e6be4430-9400-42fc-9420-fedfc1063776-chartName-val=menu_System-0&e6be4430-9400-42fc-9420-fedfc1063776-anomalies-selectedNodeIds-arr=e6be4430-9400-42fc-9420-fedfc1063776&3e65f79c-d0a6-452a-9296-2bfdaa40b2c1-anomalies-chartName-val=menu_Anomaly_advisor_submenu_Anomaly_advisor_anomaly_detection_anomaly_rate) & see if can integrate w homarr
- [ ] set up teleport (if needed)
- [x] buy separate domain name for homelab
- [x] cloudflared jellyfin migrate off of --> ???
- [ ] VMs and ansible setup
- [ ] install ddns updater and serve content using nginx proxy
- [ ] Securing Nginx Reverse proxy & open ports 
- [ ] remove old cloudflared daemons (redundancy)
- [ ] instead of exposing public IP, rent a cheap VPS, install wireguard on it and funnel traffic *from that* to home router in order to "port forward" services - see [video here](https://www.youtube.com/watch?v=7TOwr1Hs9fk)
- [ ] put mc server behind a proxy, one of [these](https://github.com/anderspitman/awesome-tunneling?tab=readme-ov-file) or minekube
- [ ] [mountainduck](https://mountainduck.io/)
- [ ] semaphore - deploy ansible & terraform scripts
- [ ] traefik (reverse proxy), portainer, semaphore stack inside portainer
- [ ] intel vtx & auto power back on after ac loss
- [ ] telstra [gen3 router root](https://github.com/seud0nym/tch-exploit?tab=readme-ov-file#type-3-non-rootable-firmware)
- [ ] DIY pfsense routers
	- [ ] [beelink mini](https://www.amazon.com.au/Beelink-Processor-3-40GHz-Computer-Business/dp/B0BYYZD6ZL?crid=1ZGBR4PINRQN&dib=eyJ2IjoiMSJ9.wfnyQTvJ8t4ly4wea-gOV4H4rCm-8jLYtcYUUxkL82T7bE1-iXqloTjOGaAI-ehbsNfMyMFmjUaYQ-pKwLXzdc9yDv0hrRIdHIcFVASAo7nGDvwUJd7rf3iHrtzO0Lam3Snn4dlH29ZTN1szp-_5aFdfHdFrHzfuiP2fMejmlF7gjlMeOYbpZVPT42N_mkkfq44KwY-s081XZ0_iXEGNpDTshOwQTCjx91Cmm-BTHtwAQvhV1Ur_FUwAqv4fQDzuAcAOiZzW2decph29lN14Hlm78srpgi7y6cHE4yzJosvSugDp011ubI6UyV8ElqklWk1nQBSNdE9ure1Oc5thO8OTMix6RVWKbUeKzlIjf2uvAod-VX59201OdP2LYETsaTDQcKyerj2fHD0Y3JancC0Qc1XOAkNjWD_y-S18AvQzPN72RIARkvMH11fpSOvH.5sZNBKHaLOF9r1q90LbQqASTDxzbtKVLGlbGdSTXECg&dib_tag=se&keywords=Beelink%2Beq&qid=1739791803&sprefix=beelink%2Bmini%2Bpc%2Caps%2C767&sr=8-2&th=1)
	- [ ] [beelink mini but a bit bigger](https://www.amazon.com.au/dp/B0CCR3DQ99/?coliid=I2JG8EQGL7F0J7&colid=34V3W8MFQHYK7&psc=1&ref_=list_c_wl_lv_ov_lig_dp_it)
	- [ ] [hardware suggestions](https://www.reddit.com/r/homelab/comments/17sbxky/small_and_silent_start_to_my_homelab_dual_nic_nuc/)
	- [ ] [more hw suggestions](https://www.reddit.com/r/homelab/comments/15ebv69/nuc_recommendations/)
	- [ ] [netgate forums](https://forum.netgate.com/topic/61878/intel-nucs-any-good-for-a-pfsense-router-build/10) 


- [x] jellyseer fix
- [x] power test draw & optimise (reply to reddit user on my post)
	- [ ] https://www.youtube.com/watch?v=MucGkPUMjNo
	- [ ] https://www.youtube.com/watch?v=zE-COCPdyEY
	In terms of power consumption, it seems to **idle at 40-50W most of the time**, with a _max_ TDP recorded of **138.6W** (under intense CPU/disk loads, i think it was when I was moving my entire media library onto it via SMB lol).

	In terms of C states, here are some of the metrics I collected with `sudo ./turbostat --show sysfs --quiet sleep 10` (see [here](https://manpages.debian.org/experimental/linux-cpupower/turbostat.8.en.html)) inside the trueNAS shell. This was at idle.
	![](posts/14/Screenshot%202025-02-15%20at%206.07.25%20pm.png)
	- [ ] 



### Hardware Monitoring with NetData:
TrueNAS ships with an instance of NetData already installed on the system, but it seems to be a very outdated version (`v1.37.1` as opposed to the latest at time of writing, `2.2.3`) with no easy way to update it apart from via the TrueNAS CLI itself. Others experienced issues with this, and the subsequent streaming of metrics to dashboards, as detailed in the terrific writeup [here](https://dizzy.zone/2024/01/20/Streaming-Netdata-metrics-from-TrueNAS-SCALE/).

As such, I opted to install another instance of NetData as a pre-configured TrueNAS app, and went from there. 


### Importing Movie Files into Radarr:
If have a bunch of stray media files, to add them all into Radarr, requires them to be in their own folders.
1. Move the miscellaneous files to the media folder monitored by Radarr & Jellyfin. For me, `/data/media/movies/`
2. Paste all the raw files in there (or better, hardlink from torrent download folder if you can)
3. To put each file in a folder with its name as a title, in `bash` run: 
   `for file in *.*; do mkdir -p "${file%.*}" && mv "$file" "${file%.*}/"; done`
4. Now in the Radarr GUI, go to "Library Import" and select that path/folder, and it should detect within it that there are now `X` number of `Unmapped Folders`. 
   ![](posts/14/Screenshot%202025-02-02%20at%207.05.54%20pm.png)
   Click on the path, and then Radarr will attempt to auto-guess what each movie is based on the filename, which may require a bit of manual cleanup but is doable.
   ![](posts/14/Screenshot%202025-02-02%20at%207.06.10%20pm.png)
   Then, click `Import Movies` and watch as they magically are mapped into your library!

### Setting up TVDB in Jellyfin

As sonarr uses
- Install [TVDB Plugin](https://github.com/jellyfin/jellyfin-plugin-tvdb), searchable within Jellyfin Admin Panel - `Catalog`
- Set TVDB as priority for all metadata download within the "Shows" folder
  ![](posts/14/Screenshot%202025-02-02%20at%201.15.46%20pm.png)
- Select the show you want to refresh, erase all existing metadata by selecting `Edit Metadata` and then `Reset`. Give it a new name, ensure it has no external IDs linked to it, and ensure the folder name has the TVDBID in it (e.g. `[tvdbid-83915]`). Then `Save`.
  ![](posts/14/Screenshot%202025-02-02%20at%201.19.53%20pm.png)
- Then, click `Refresh Metadata` and select the following. Now, Jellyfin will use the folder name's `tvdbid-83915` to search and correctly identify it on the [TVDB.](https://thetvdb.com/) 
  ![](posts/14/Screenshot%202025-02-02%20at%201.21.07%20pm.png)
  I mention doing it this way instead of via the plugin's built in IDs, as they can be quite particular about what is needed for it to detect & match properly. For example, just using the folder name `[tvdbid-83915]` sufficed, and found all the below IDs and slug IDs. 
  ![](posts/14/Screenshot%202025-02-02%20at%201.22.50%20pm.png)
  Alternatively, you can "Identify" the show and search directly by setting `TheTVDB Numerical Series Id = 83915`, but I found this to be a bit spotty at times. 
  ![](posts/14/Screenshot%202025-02-02%20at%201.25.59%20pm.png)



### Setting up SSL Cert for homelab (NGINX Proxy manager, Cloudflare)
- [Wolfgang's video](https://www.youtube.com/watch?v=qlcVx-k-02E&t=594s)
- [TrueNAS Core-specific guide](https://wiki.familybrown.org/en/fester/configure-apps/other/npm)
	- needs [PiHole](https://www.youtube.com/watch?v=yp1AbZlIHhM) (local DNS entries)
	- how its done - https://www.youtube.com/watch?v=nmE28_BA83w&t=877s
	  


### Jellyfin Configuration:

#### Use an SSD as a cache drive & store cache/transcoding/config on an SSD to improve performance! 
Do **NOT** map these cache & transcoding paths to a HDD (likely where your media files are stored). Pass the HDD through as a separate storage folder for your media, and map the cache/transcoding/config paths to an **SSD on your Host**. 

This is because when it comes to transcoding, r/w operations are much higher as blocks are constantly being written to and erased from these locations as the stream of data is transcoded & played back in Jellyfin.

![](posts/14/Screenshot%202025-01-31%20at%209.53.44%20pm.png)
Having a dedicated SSD cache drive in your TrueNAS ZFS pool will help with this too (I imagine)
![](posts/14/Screenshot%202025-01-31%20at%2011.47.10%20pm.png)

### Hardware encoding with Jellyfin:
I'm using an Intel 12th-gen 12400 with an iGPU that has QSV enabled.
To pass this through to jellyfin, ensure that the GPU passthrough is ticked on the app install page, or you'll get fatal errors when `ffmpeg` can't access the driver.
![](posts/14/Screenshot%202025-01-31%20at%2011.10.26%20pm.png)

Once in the Jellyfin UI, go to **Admin -> Dashboard -> Playback -> Transcoding**, and enable Hardware Acceleration for Intel Quicksync. ![](posts/14/Screenshot%202025-01-31%20at%2011.37.50%20pm.png)
Check where your iGPU is located using the TrueNAS CLI, but (as stated) it's likely at `/dev/dri/renderD128` (it was for me).
Before proceeding, follow the [additional configurations for Hardware acceleration](https://jellyfin.org/docs/general/administration/hardware-acceleration/) for your (i)GPU. This took a while to read through, but was well worth doing. 

Begin with verifying the available GPU (in the TrueNAS CLI) the `lspci` command:
 `lspci -nn | grep -Ei "3d|display|vga"`

To see [whether it's safe to enable low power decoding](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel/#configure-and-verify-lp-mode-on-linux) in the Jellyfin UI, pop back to the TrueNAS CLI and run the following commands:
  1. `lspci -knn | grep -E "i915|xe|VGA|Display"`
     ensure the `i915` (or `xe`, see [here](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel/#configure-and-verify-lp-mode-on-linux)) driver is in use:
     ![](posts/14/Screenshot%202025-01-31%20at%2011.08.20%20pm.png)
  2. reboot, then check the GuC & HuC status with the following commands, **make sure there is no FAIL or ERROR in the outputs**:
     `sudo dmesg | grep -E "i915|xe"`
     `sudo sh -c "cat /sys/kernel/debug/dri/0/gt*/uc/guc_info"`
     `sudo sh -c "cat /sys/kernel/debug/dri/0/gt*/uc/huc_info"`


For the rest of the settings in the Jellyfin UI, I left most as default, but set the rest according to what was recommended in the [docs here for my QSV iGPU](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel#transcode-h264) (toggling off `H264`, `AVI` and `VP8`), in line with what hardware encoding & decoding [my processor supported](https://www.intel.com/content/www/us/en/docs/onevpl/developer-reference-media-intel-hardware/1-1/overview.html#DECODE-OVERVIEW-11-12). 
![](posts/14/Screenshot%202025-01-31%20at%2011.37.18%20pm.png)
![](posts/14/Screenshot%202025-01-31%20at%2011.16.24%20pm.png)
Please adjust according to your individual needs (I will be doing so too as I experiment).

Then, scroll down to the bottom, press `Save`, and restart jellyfin just to be safe. 
Then, open any video & test. Since I enabled hardware decoding for `HEVC`, I selected a video in that format, and began playing it.
![](posts/14/Screenshot%202025-01-31%20at%2011.18.03%20pm.png)
I then could check that it was indeed transcoding via selecting `Playback Info` in the bottom left hand corner, and ensuring the play method said `Transcoding`.
![](posts/14/Screenshot%202025-01-31%20at%2011.18.39%20pm.png)


### Making Jellyfin & Jellyseer accessible to the outside world!
I figured there were three main ways to do this:
1. Manual port forwarding on my router (ew, not secure at all)
2. Using Cloudflare Tunnel to create what is essentially a reverse tunnel between the `Cloudflared` daemon (likely running on your server) and Cloudflare's servers, connected to a domain name that you own. Yes, this has a number of privacy and a few smaller security implications (see video [here](https://www.youtube.com/watch?v=oqy3krzmSMA&t=364s), for example) but it *is* zero config, the jellyfin is behind a login page and Cloudflare deals with encryption/HTTPS & can set custom layer 7 access policies if you're *really* paranoid. Not perfect, but it works.
![](posts/14/Screenshot%202025-02-02%20at%209.09.44%20pm.png)
3. **Accessing the network through Tailscale** - either to traverse it locally (for management, this is what I do via a Tailscale subnet router installed on a separate proxmox device), or expose a specific device/service on the NAS by installing the Tailscale TrueNAS app & going from there. However, this **requires some client-side configuration** (being downloading a Tailscale client and authenticating to that before *then* logging into the Jellyfin server), and was deemed a *little* overkill for just sharing with friends.

Thus, my process was as follows:
#### Cloudflared with TrueNAS Setup
1. Open your Cloudflare account (associated with the domain name you control) & navigate to [Network > Tunnels](https://one.dash.cloudflare.com/networks/tunnels).
2. Curse at the world when you are reminded to "enable the FREE feature" through their stupid "Checkout" before proceeding (redundancy at its best, but okay).
3. Press "Create Tunnel", then select `Cloudflared` as the type, and name it for the service that you're remotely accessing..
   ![](posts/14/Screenshot%202025-02-02%20at%209.19.40%20pm.png)
   Select `Docker` as the means of install, and copied the token ID specified in the connector install section.
   ![](posts/14/Screenshot%202025-02-02%20at%209.20.55%20pm.png)
4. Go to TrueNAS and install the `Cloudflared` application (or start another instance of one). During setup, enter the token from the previous step, and that's literally it! After its deployed, it should pop up as a "connecter" back on your Cloudflare dashboard after a few minutes! If not, check the logs in TrueNAS for the `Cloudflared` application, to ensure it hasn't errored out.
   ![](posts/14/Screenshot%202025-02-02%20at%209.24.20%20pm.png)
   ![](posts/14/Screenshot%202025-02-02%20at%209.27.45%20pm.png)
6. Click `Next`, and then add the details for the service you want the Cloudflared daemon to tunnel to. For me, this was Jellyfin (port `8096` on my NAS) on one daemon, and Jellyseer for requests (port `5055` on my NAS) on another - each mapped to a different subdomain.
   ![](posts/14/Screenshot%202025-02-02%20at%209.30.19%20pm.png)
   Then, just `Save tunnel`, and navigate to the URL you specified! (e.g. from above, `requests.XYZ.com`)
7. Voila! Two remotely accessible services served fresh from your homelab, each protected by a built-in login with no exposing your home IP!
   ![](posts/14/Screenshot%202025-02-02%20at%209.36.13%20pm%20Large.jpeg)
   ![](posts/14/Screenshot%202025-02-02%20at%209.36.51%20pm.png)
Now just create users for your friends and assign them the appropriate requesting & watching privileges!

### Cloudflare Part 2: There be Trouble (TOS):

### Setting up Nginx Reverse proxy



### Securing Nginx Reverse proxy & open ports

(https://www.grc.com/shieldsup)
![](posts/14/Screenshot%202025-02-15%20at%2010.08.31%20am.png)






### UptimeKuma
Hosted on a separate machine (proxmox) in order to continue monitoring if TrueNAS itself goes down.
- Running in docker, inside a LXC Ubuntu 24.04 container
- Install commands:
	- install [docker](https://docs.docker.com/engine/install/ubuntu/)
	- install & run [uptimekuma](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install) as a container:
	  `docker run -d --restart=unless-stopped -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1`
	- set a static IP for the LXC to make accessing it easier (locally)
- Once into the webUI, I repeated the below process to add my various services running on the TrueNAS Scale, simply adjusting the `port` & `Friendly Name` field to match those used by the given application.
  ![](posts/14/Screenshot%202025-01-31%20at%206.44.52%20pm.png)
I can then set up custom notifications (email, webhook, discord, *anything*) to be sent if any of these services fail these ongoing health checks, enabling me to troubleshoot and understand any potential downtime in the future.
![](posts/14/Screenshot%202025-01-31%20at%206.39.52%20pm.png)
This is just scratching the surface of what this application can do - see how many request types it supports below - and I look forward to playing around with it more in the future!
![](posts/14/Screenshot%202025-01-31%20at%206.48.25%20pm.png)
> **Future plans - see [here](https://www.youtube.com/watch?v=r_A5NKkAqZM)**


### Auto-downloading new Torrents with Autobrr:
- create a search feature to auto-grab NEW torrents on TL (new ones) that have between 1-30 seeders and 20+ leechers --> use [autobrr](https://wiki.torrentleech.org/doku.php/autobrr?s[]=rss)
- installed via trueNAS app, mapped to port 7474, followed [torrentleech guide](https://wiki.torrentleech.org/doku.php/autobrr) 
- [Filter examples (for radarr, sonarr, only episodes, etc.)](https://autobrr.com/filters/examples)
- Created following filters:
![](posts/14/Screenshot%202025-02-05%20at%2010.40.07%20pm.png)
- in settings, connect radarr & sonarr as download clients (usual story - paste API key & set the radarr/sonar IP & port being used. 
![](posts/14/Screenshot%202025-02-12%20at%208.52.56%20pm.png)
then, go to "lists" feature and you can set any monitored movies within sonarr/radarr be added to your existing filters in autobrr. May be a *bit* redundant, but I feel like i've got my bases covered in this way should radarr/sonarr fail to find it quick enough with its own auto-find feature after its released. ![](posts/14/Screenshot%202025-02-12%20at%208.51.47%20pm.png)
![](posts/14/Screenshot%202025-02-12%20at%208.52.02%20pm.png)


## VMs and ansible setup



### Teleport to act as a proxy for remote, secure access & auth to internal services:
- https://goteleport.com/docs/installation (in docker on TrueNAS)
- https://goteleport.com/docs/get-started/ (teleport server config & setup)
- [video guide here](https://www.youtube.com/watch?v=iNuQbqtKvfs&t=490s)

## Anonymity, Security & Privacy
### Setting up a SOCKS5 Proxy:
**Why?** Hides your "home" IP by redirecting all traffic through a proxy server (& IP), typically owned by a VPN/VPS. Can direct your torrent client ([qbittorrent](https://www.qbittorrent.org/) for me) to use it by entering the proxy server address, port, username, and password (provided by your proxy service) into it.
![](posts/14/Screenshot%202025-01-27%20at%204.29.38%20pm.png)

**VPN vs SOCKS5 Proxy:** The key difference is that SOCKS5 proxy traffic only **masks your IP**, and does **not encrypt/tunnel your traffic with Wireguard/OpenVPN/IPSec**, like a VPN does. This means that SOCKS5 proxies are much faster, and often useful for hiding you from any immediate prying eyes in any torrent swarm you become part of. Can be thought of as just replacing your home IP with the one of a house down the street - yes, it *could* be traced back to you should that IP be compromised & the owner interrogated, but it can provide an extra layer of anonymity.

- [Guide here (PIA)](https://helpdesk.privateinternetaccess.com/kb/articles/do-you-offer-a-socks5-proxy)

