---
title: Minecraft Server From Scratch (Proxmox LXC, Docker Compose + itzg)
date: 2025-04-13
description: spinning up a minecraft server with docker-compose in proxmox
toc: true
math: true
draft: false
categories: proxmox
tags:
  - minecraft
  - server
  - docker
  - docker-compose
  - lxc
  - itzg
---

Just a smol lil guide for myself to set up a minecraft server from scratch, *as I cannot count the number of times I've had to re-learn this when I migrate from server to server*. 

I've opted for services that *should* (for the most part) be supported long-term and are relatively secure & lightweight. However, as any good netizen should do, ***please take my advice with a granule of sugar***...

## - Set up `LXC` Container in Proxmox
1. Create new container in `proxmox` using the `Ubuntu 24.04` `LXC` image (or your desired flavour, noting commands may differ slightly depending on package managers) - allocating at least `4GB RAM` & `2-4 CPU` cores to the machine.
2. Once logged in, create a [sudo-enabled user](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu) with:
   `adduser myuser`
   Set the password, then:
   `usermod -aG sudo myuser`
   `su myuser`
3. **Harden SSH** - ensuring the following values are changed & set within `/etc/sshd_config`. 

```bash
Port 22
PermitRootLogin no
MaxAuthTries 4
MaxSessions 2

PubkeyAuthentication yes

PasswordAuthentication no
PermitEmptyPasswords no

KbdInteractiveAuthentication no

UsePAM no

X11Forwarding no
PrintMotd no
ClientAliveInterval 600
ClientAliveCountMax 2
```

3. Add your local machine's `ed25519_pub` key to the `~/.ssh/authorized_keys` file (creating it, if it doesn't exist). This will allow key-based login for user `myuser`. 

***Be careful not to lock yourself out here, test with password based login first! For example, by setting `PasswordAuthentication yes` and logging in, before changing it to `PasswordAuthentication no`***.

4. Ensure `DNS` is setup properly - check `/etc/resolv.conf`. 
Basic internet functionality can be tested & achieved by having the line `nameserver 8.8.8.8`, but configure to your use case.

5. Login from local machine with `ssh myuser@XXX.XXX.XXX.XXX`. Test your sudo privileges with `sudo ls /root`.

6. Lock root account with `sudo passwd root -l`.

7. Run `sudo apt update && sudo apt upgrade -y`.

8. Find the current `IP` with `ip -a` (typically on the `eth` interface) and set it as static (in `proxmox` and/or on your router).


## - Install Docker (Compose) & itzg Minecraft Server

1. **Install Docker Engine** - follow steps (distro-specific) [here](https://docs.docker.com/engine/install/), as you will need to configure your package repository properly.

2. **Install Docker Compose** - following steps [here](https://docs.docker.com/compose/install/linux/#install-using-the-repository). `sudo apt install docker-compose` worked for me.

3. **Make a new directory for the Minecraft server** to sit in: `~/minecraft`.

4. Inside, **create a `docker-compose.yml`**, generated with something like [setupmc.com](https://setupmc.com/java-server/) to specify server version, plugins, etc. 

My example `docker-compose.yml` file is below *(for a `1.18` server, replacing Timezone (`TZ`) accordingly)*:

```yaml
services:
	mc:    
		image: itzg/minecraft-server:java17    
		tty: true    
		stdin_open: true    
		ports:       
			- "25565:25565"     
		environment:       
			EULA: "TRUE"       
			TYPE: "PAPER"       
			VERSION: "1.18"       
			PAPER_CHANNEL: "experimental"       
			MEMORY: "4096M"       
			MOTD: "welcome, traveller, to an older time...
			USE_AIKAR_FLAGS: "true"       
			TZ: "[YOUR-TIMEZONE]"     
		volumes:       
			- "./data:/data"
```

5. **Start the container** from within the same directory as `docker-compose.yml` with `sudo docker compose up -d`. After the image is finished being pulled from the [itzg minecraft server repo](https://github.com/itzg/docker-minecraft-server/tree/master), **watch the logs** as the server starts with `sudo docker compose logs -f`. 

6. *If you get an error message about the “class file version” after starting the server, check [this table to see which Java version corresponds to the respective class file version](https://setupmc.com/guides/determining-correct-java-version-for-operating-minecraft-server/). Then adjust the Docker image tag in the setupmc.com](https://setupmc.com/java-server/) configurator accordingly.*

7. **To stop the server**, run `sudo docker compose down`.

8. **To migrate a world save file over** (if applicable), copy the following files (at minimum) over from your old server (using something like `scp`, or via a GUI if you install something like [webmin](https://webmin.com/download/)):
    - `server.properties`
    - `/world`
    - `/world_the_nether` (if exists)
    - `/world_the_end` (if exists)
    - `whitelist.json` (if applicable)

*Ensure to tweak server-specific configurations within `server.properties` if needed!*

9. As you're running through `docker`, it should handle the local network ports on the `lxc` for you nicely (if on a fresh `linux` install). Also, before I continue, it would be remiss of me to exclude the obligatory ***do this so at your own risk, and please consider the below server hardening methods:
   - [not running the server as root! *(not a problem if you followed the guide above)*](https://madelinemiller.dev/blog/root-minecraft-server/)
   - [general server tips](https://madelinemiller.dev/blog/ultimate-guide-running-minecraft-server/#security) & [links to hardening methodology](https://www.spigotmc.org/threads/minecraft-security-part-1-awareness.414081/)

9. With that out of the way, now time to **open up a port on your local** router/modem. For me, I've opted for a little "security through obscurity" (a contentious topic, but given my threat model) by mapping my **router's external port**, `43456` to the default minecraft listening port (`25565` - specified in `server.properties`) on my `lxc` machine:
    
    ![](/posts/14/Screenshot%202025-04-13%20at%209.15.24%20pm.png)
	Additionally, I've set up a `DNS A record` for the domain I own to point at my router's `public IP`, so I can access my server (and share it) with `my-domain:43456`.

***Now, you should be all up and running! :3***

<div style="text-align: center">
   <img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExYTY3OTM0YTI1dzBtb2htNHRzbzdtaTA1Yzh4ZHQ2aXdmZmZqb2xscCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/a6pzK009rlCak/giphy.gif" style="max-width: 400px; max-height: 300px"></img>
</div>


## - For any further troubleshooting
- [itzg Docker Minecraft Server Documentation](https://docker-minecraft-server.readthedocs.io/en/latest/#using-docker-compose)
- [`docker-compose.yaml` configuration generator (SetupMC)](https://setupmc.com/java-server/)