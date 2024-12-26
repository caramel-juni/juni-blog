---

title: "'securely' setting up web server with nginx @ home & self hosting"
date: "2024-12-26"
description: "don't do this it ain't worth it"
toc: true
math: false
draft: false
categories:
    - linux
    - nginx
tags:
    - self hosting
    - debian
    - nginx proxy manager
    - ufw
    - docker
    - docker-compose

---


1. debian 12 container install inside proxmox
2. `sudo apt update && sudo apt upgrade -y
3. network settings: If your router supports subnets/VLANs, connect this to the isolated VLAN. within proxmox, assign static IP not in use and point to your router's gateway.![[Screenshot 2024-07-08 at 8.40.32 PM.png]]
4. install nginx on debian - `sudo apt install nginx -y`
5. create a file for website settings: `nano /etc/nginx/sites-available/mywebsite
```nginx
server {
        listen 80 ; 
        listen [::]:80 ;
        server_name juni-mp4.org ;
        root /var/www/juni-web ;
        index.html index.htm index.nginx-debian.html ;
        location / {
                try_files $uri $uri/ =404 ;
        }
}
```
The `listen` lines tell `nginx` to listen for connections on both IPv4 and IPv6.
The `server_name` is the website that we are looking for. By putting `landchad.net` here, that means whenever someone connects to this server and is looking for that address, they will be directed to the content in this block. `root` specifies the directory we're going to put our website files in. 

This can theoretically be wherever, but it is conventional to have them in `/var/www/`. Name the directory in that whatever you want. `index` determine what the "default" file is; normally when you go to a website, say `landchad.net`, you are actually going to a file at `landchad.net/index.html`. That's all that is. Note that that this in concert with the line above mean that `/var/www/landchad/index.html`, a file on our computer that we'll create, will be the main page of our website. 

Lastly, the `location` block is really just telling the server how to look up files, otherwise throw a 404 error. Location settings are very powerful, but this is all we need them for now.
7. create directory for your website's contents/files using: `mkdir /var/www/juni-web` (can be located wherever but standard to store in `/var/www/[X]` ) where you can place website files like `index.html` etc.)
8. enable the site by making a link between the config file in you just created in `sites-available` and the `sites-enabled` directory: 
	`ln -s /etc/nginx/sites-available/juni-web /etc/nginx/sites-enabled/
9. restart nginx `systemctl restart nginx`

***make sure the "default" file doesn't remain in `/etc/nginx/sites-enabled/` otherwise will serve the default config page for nginx!!***


## Main Nginx Files & Explanation:
*The idea is that you can make a site configuration file in `sites-available` (that links to where your website is stored locally, e.g. `/var/www/sitestorage`), then make a link to this configuration file in `sites-enabled`, which will activate it.* 

### Config Files:
- `/etc/nginx/sites-available/` - directory containing any site configuration files. Points to directory containing main website content, e.g. `/var/www/juni-web`
	```nginx
	server {
	        listen 80 ;
	        listen [::]:80 ;
	        server_name juni-mp4.org ;
	        root /var/www/juni-web ;
	        index.html index.htm index.nginx-debian.html ;
	        location / {
	                try_files $uri $uri/ =404 ;
	        }
	}
	```

- `/etc/nginx/sites-enabled/` - directory containing **links** to site configuration files
	make links via: `ln -s [link-source-path] [link-destination-path]

### Main website location:
- `/var/www/[site-name]`'
	e.g. `/var/www/juni-web`
	contains files like index.html, etc.



# Securing it:

### UFW:
sudo apt install ufw

```
# Limit SSH access to port 22 
sudo ufw limit 22/tcp 

# Allow HTT![[Screenshot 2024-07-19 at 9.04.25 PM.png]]P traffic on port 80 
sudo ufw allow 80 

# Allow HTTPS traffic on port 443 
sudo ufw allow 443 

# Limit SSH access to port 22 for IPv6 
sudo ufw limit 22/tcp6 

# Allow HTTP traffic on port 80 for IPv6 
sudo ufw allow 80/tcp6 

# Allow HTTPS traffic on port 443 for IPv6 
sudo ufw allow 443/tcp6

ufw enable

ufw logging on

ufw status

```

![[Screenshot 2024-07-09 at 11.51.31 PM.png]]
https://www.linode.com/docs/guides/configure-firewall-with-ufw/



# docker install ([debian](https://docs.docker.com/engine/install/debian/)):
Run the following command to uninstall all conflicting packages:
```bash
 for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

install dependencies:

```bash
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

Set up Docker's `apt` repository.

``` bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

install latest docker version
```bash
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Verify that the installation is successful by running the `hello-world` image:

```bash
 sudo docker run hello-world
```


## docker compose install

why install it? manage all containers & deployments from a [single yaml file](https://docs.docker.com/compose/) 

``` bash
sudo apt-get update

sudo apt-get install docker-compose-plugin

docker compose version

```
![[Screenshot 2024-07-10 at 12.12.09 AM.png]]

create compose file near website data for ease of management 
```bash
## if website located in mkdir /var/www/juni-web
mkdir /var/www/docker-compose
nano docker-compose.yml
```
we can use this to install...

### nginx proxy manager (NPM) install 
*(not to be confused with node package manager npm lol)*

*note: make sure to set ports for managing nginx proxy manager (NPM) to 8080 & 4443 (or whatever custom ones you'd like) and NOT 80 & 443, as the latter will likely be in use by nginx to serve & access your website at.*

in the docker-compose.yml...

``` bash

cd /var/www/docker-compose
nano docker-compose.yml

## then add into file:
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '8080:80' # Port for HTTP access to NPM
      - '4443:443' # Port for HTTS access to NPM
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    # Uncomment the next line if you uncomment anything in the section
    # environment:
      # Uncomment this if you want to change the location of
      # the SQLite DB file within the container
      # DB_SQLITE_FILE: "/data/database.sqlite"

      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt


## then run

docker compose up -d
```

access nginx via `http://[server-ip]:81` & login with `admin@example.com` and `changeme`  (changed upon entry)


## cloudflare setup

sign up for free cloudflare account
follow signup steps to point existing domain at cloudflare

autoscan for any DNS records you changed with your registrar (* domains, subdomains etc.) so cloudflare is aware of them

![[Screenshot 2024-07-19 at 9.05.00 PM.png]]
![[Screenshot 2024-07-19 at 9.09.40 PM.png]]

navigate to your domain registrar and set the custom DNS servers to the ones provided to you by cloudflare.

![[Screenshot 2024-07-19 at 9.08.37 PM.png]]

cloudflare setup guide here - https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/
![[Screenshot 2024-07-19 at 9.17.23 PM.png]]
![[Screenshot 2024-07-19 at 9.17.34 PM.png]]
![[Screenshot 2024-07-19 at 9.18.03 PM.png]]


API token:  HRWvk067sLPv_RMGDPhS1y0lj5XDcLErat5nY18m
verify with cul command:
`   curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \        -H "Authorization: Bearer [YOUR TOKEN]" \ -H "Content-Type:application/json"`


### Cloudflare & SSL issues (certbot)
if you've setup certbot or something similar to manage ssl certificates on your nginx server, MAKE SURE to go to cloudflare and select Full (strict) SSL/TLS encryption mode so it doesn't have an SSL mismatch and make your site inaccessible via the browser - ![[Screenshot 2024-07-19 at 10.00.00 PM.png]]


**The Why:** as with **'flexible'** ticked, cloudflare will (by default) try and make requests to your server via HTTP and the server will throw an error if it's using SSL due to a cipher mismatch, then browsers interpret this as a potential MiTM attack. see below: ![[Screenshot 2024-07-19 at 10.03.20 PM.png]]
![[Screenshot 2024-07-19 at 10.03.46 PM.png]]

you can also check your site's nginx config file to see that certs are set up properly:

![[Screenshot 2024-07-19 at 10.07.33 PM.png]]

## OPENING the ports

**External port**: what port is used by external users to access, like:
`pu.bl.ic.ip:[external-port]`
e.g. `182.46.382.83:443`

**Internal port**: what port on the specified **Device** (the one identified by the `Device IP Address` field) that the traffic will be forwarded to.

![[Screenshot 2024-07-19 at 10.24.08 PM.png]]


![[Screenshot 2024-07-19 at 10.23.53 PM.png]]

## set up static IP for container in proxmox on router
OR just change the DHCP pool to not include the IP address you want statically added on the proxmox

(e.g. setting DNS pool to `192.168.0.20` ->  `192.168.0.200` and then assigning static IP for your container in proxmox outside of the pool range but on the same subnet, e.g. `192.168.0.5` )
![[Screenshot 2024-07-19 at 10.19.57 PM.png]]

![[Screenshot 2024-07-19 at 10.19.45 PM.png]]


## adding SSL cert to nginx proxy manager

![[Screenshot 2024-07-19 at 10.40.02 PM.png]]

certs on web server:
![[Screenshot 2024-07-19 at 10.43.38 PM.png]]


## setup proxy host on NPM


![[Screenshot 2024-07-19 at 11.37.22 PM.png]]
![[Screenshot 2024-07-19 at 11.38.09 PM.png]]


## setup NPM & dynamic DNS


to do:
- [ ] https://anebula.io/how-to-set-up-nginx-proxy-manager-using-docker-compose/
- [ ] https://www.youtube.com/watch?v=GarMdDTAZJo
- [ ] https://notthebe.ee/blog/easy-ssl-in-homelab-dns01/
- [ ] set up nginx reverse proxy, cloudflare etc. https://blog.prutser.net/2021/01/20/how-to-securely-self-host-a-website-or-web-app/
- [ ] install certbot & auto renewal & setup https
- [ ] setup firewall around docker - https://docs.docker.com/network/packet-filtering-firewalls/#docker-and-ufw
- [ ] ssh harden copy config files & replace keys
- [ ] install auto updates for all respective software (docker, docker compose, nginx, nginx proxy manager, ufw, anything else used)
- [ ] port forward website to internet to make accessible
- [ ] update domain registrar to point to local public IP
- [ ] write scp command that writes locally-edited files to website remotely
	`scp -r user@[remoteTargetComputerIP]: [RemoteFilesPath] [localDestinationPath]
	e.g. `scp -r root@45.77.26.67:/var/www/mysite ~

---

**DISCLAIMER:** *I would consider this a LEGACY POST of mine, written a long time ago. Please excuse any typos, errors or lapses in memory/judgement - as it was added to the site from the archives, just to put everything in one place. Thankq for your understanding 🙇‍♀️*

---
