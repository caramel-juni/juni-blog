---
title: ""
date: 2025-02-12
description: ""
toc: true
math: true
draft: true
categories: 
tags:
---

### Setup New VM, harden SSH, add authorized keys

``` bash
'
THE BELOW SCRIPT SETS UP THE FOLLOWING ON A DEBIAN BASED DISTRO:
- SSH ACCESS + AUTHORIZED KEYS
- ufw

LINES TO CHANGE BASED ON ENVIRONMENT

line 65 - ssh key (insert yours)
lines 71 onwards --> uncomment to install tailscale and set up as an exit node/subnet router, need to replace with desired IP etc.

'
```

First, run the bellow commands MANUALLY to create a root acc without pw
`sudo passwd -d root
`su root`

Then, run the following script **as root or sudo-enabled user**.
Ensure the script begins with `#!/bin/bash` and can be executed by running a command like `chmod 744` (only read/write/executable by the owner, root. one of many potential permission sets, the important thing is to have 7 up first so you, the owner, can run it).
Then, run the script with `./scriptname.sh` from the folder the script is in.
``` bash
#!/bin/bash

## THE BELOW ASSUMES YOU ARE RUNNING AS ROOT USER. 

## --------------------------
## INSTALL REQUIRED PACKAGES
## --------------------------


sudo apt update && sudo apt -y upgrade && sudo apt -y autoremove && sudo apt clean


## ---------------
## Setup UFW
## ---------------

ufw limit 22/tcp
ufw limit 22/tcp6
ufw enable
ufw logging on
ufw status


## ---------------
## Harden SSH
## ---------------

sudo sed -i -e 's/#PasswordAuthentication yes/PasswordAuthentication no/' \
		-e 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' \
        -e 's/UsePAM yes/UsePAM no/' /etc/ssh/sshd_config
        -e 's/#PermitUserEnvironment no/PermitUserEnvironment no/'
		

## remove any conflicting settings for password auth
sudo sudo rm -rf /etc/ssh/sshd_config.d/*


## write known good SSH key to the authorized_keys file. REPLACE WITH YOUR SSH PUBLIC KEY (.pub file) generated when using ssh-keygen (its contents begin with "ssh-rsa AAAAB3...")

sudo echo "ssh-rsa [key]= [usr]@[domain/hostname]" >> ~/.ssh/authorized_keys

## Lock the root account
passwd -l root
```

### Setting up NGINX web server:

First, re-enable the root user if you disabled it, or run with sudo.

``` bash
sudo passwd -d root
# or if a root account exists:
sudo passwd -u root

su root
```

Then create & run the following `.sh` file:

``` bash
#!/bin/bash

## THE BELOW ASSUMES YOU ARE RUNNING AS ROOT USER. 

apt update && apt -y upgrade && apt -y autoremove && apt clean

## if serving over a domain name, use certbot & python3-certbox-nginx. not needed over IP

apt install nginx certbot python3-certbot-nginx -y

nano /etc/nginx/sites-available/mywebsite >> 

server {
        listen 80 ; 
        listen [::]:80 ;
        server_name base-site.com ;
        root /var/www/base-site ;
        index index.html index.htm index.nginx-debian.html ;
        location / {
                try_files $uri $uri/ =404 ;
        }
}

ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled

rm /etc/nginx/sites-enabled/default


mkdir /var/www/base-site

cat <<EOF > /var/www/base-site/index.html

<!DOCTYPE html>

<h1>Welcome to your small corner of the interweb!!</h1> 

<p>This is just the beginning... a blank canvas to fill with code!</p>

<p>You got this! <3 </p>

EOF
```
Then, depending on whether you need to install ufw:
``` bash
apt install ufw
```
Then:
``` bash
#!/bin/bash

## THE BELOW ASSUMES YOU ARE RUNNING AS ROOT USER. 

sudo ufw limit 22/tcp 
sudo ufw allow 80 
sudo ufw allow 443 
sudo ufw limit 22/tcp6 
sudo ufw allow 80/tcp6 
sudo ufw allow 443/tcp6
ufw enable
ufw logging on
ufw status

systemctl restart nginx


# lock root account

passwd -l root

```

If serving over a domain name, as letsencrypt doesn't do IP addresses, run the following:
``` bash
## grabs the machine's hostname/IP(s) (excluding localhost) then uses awk to extract & print the FIRST field from the output. stores this in a variable

## e.g. if IPs returned by hostname -I were 192.168.1.100 10.0.0.5, it would return ONLY 192.168.1.100.

## this IP is passed into the `certbot --nginx -d` command as the hostname to issue the ssl certificate for (meaning of the -d flag)

sudo certbot --nginx -d $(hostname -I | awk '{print $1}') --non-interactive --agree-tos --register-unsafely-without-email --deploy-hook "systemctl reload nginx"

```