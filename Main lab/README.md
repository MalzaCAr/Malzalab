# Main Homelab Server
Here are the services and configs that my most powerful "server" is running.
This "server" is really just an old gaming rig that I gave a new life to after it's hardware became basically obsolete.
## Specs
### CPU: Intel Core i3 6100
### GPU: Nvidia GTX 1650 Super 4gb
### RAM: 8GB DDR4 2133 MHz (desperately needs an upgrade)
### Storage:
- 1x 256Gb SATA SSD boot drive
- 2x 1TB whatever ancient HDDs running RAID 1
# Overview of stuff running here
- [Docker](#docker)
  - [Portainer](#portainer)
  - [Watchtower](#watchtower)
  - [Immich](#immich)
  - [Vaultwarden](#vaultwarden)
  - [Omnitools](#omnitools)
  - [Jellyfin](#jellyfin)
  - Arr* stack
- Nginx reverse proxy
  
# Docker
Most of the stuff I actually use (with a few exceptions like Pi-hole) run via docker and docker compose. If you've never used docker, there are plenty of tutorials online, but to use my config, 
all you have to do is install docker from [here](https://docs.docker.com/engine/install/), 
and either copy a `docker run` command, or navigate to a folder with a `docker-compose.yml` file and type in `docker compose up -d`.
For more information, use the interwebs or [check the docs](https://docs.docker.com/).

## Portainer
Portainer is a docker management tool where you can manage images, stacks, containers and other stuff. To install it, I followed the [portainer docs](https://docs.portainer.io/start/install-ce/server/docker/linux), but if you're lazy, 
basically run 

```
docker volume create portainer_data` to create a volume needed by docker, then run `docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```

After that, access the webui at `https://localhost:9443.`

## Watchtower
Watchtower is a tool that automatically scans and updates docker images for all of your current active docker containers.
<br/>
**DISCLAIMER: I YOLO MY UPDATES. THERE ARE SMARTER WAYS TO DO THIS.**
<br/>
You can find it's docs [here](https://containrrr.dev/watchtower/), but I use this command to run it:
```
docker run --detach \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    -v /etc/localtime:/etc/localtime:ro \
    containrrr/watchtower \
    --cleanup \
    --schedule "0 3 * * *"
```
With this configuration, the updates will run every day at 3 AM. You can edit the `schedule` argument to fit whatever needs suit you, using the standard cron syntax.

## Immich
Immich is a google photos alternative for accessing your photos online and backing them up.
There are a few things special about my setup. Alongside the `docker-compose.yml` file, you'll find some extra files:

#### hwaccel.ml.yml
This is a file used to configure hardware acceleration for Immich's machine learning, used for stuff like facial recognition. You can use this file if you have an Nvidia GPU, but otherwise you'll have to adjust some variables in the file.

#### hwaccel.transcoding.yml
Similar story to the other hardware acceleration file, just in this case used for transcoding.

### restore.sh
This is a shell script used to restore Immich's database from a database dumps. Super TL;DR, Immich has a library folder where all of your photos are stored, but also a **database** in the backend used to tell immich where the files are actually located, and I presume for performance reasons. **Immich will not recognize your library if the database is missing**.
Immich does automatic database dumps every night at 2AM, so if something happens to your Immich installation, you use that database dump along with the restore.sh script to restore immich's database.

#### Library template
While on the topic of libraries, I would highly recommend to set a `Storage template` once you have Immich up and running. If omitted, Immich will place the photos however it likes into folders with garbage and nonsensical names.
You can set the storage template either in setup, or in administration settings.
This is what I use:
```
{{#if album}}{{album}}{{else}}Other{{/if}}/{{filetypefull}}/{{filename}}
```
This will organize your photo library by album (or placing in "Other" if no album), and into seperate IMAGE or VIDEO folders.
<br/><br/>
For more info about all of this, check out the [docs](https://immich.app/docs/overview/welcome/)

## Vaultwarden
Vaultwarden is a password manager that's compatible with **Bitwarden**.
The setup is pretty simple: run docker compose, go to the webui, set it up, then you can integrate it into a Bitwarden extension/app.
I aditionally set up `Vaultwarden backup`, that does daily backups and places them into the /backup folder. I plan to use this with rsync to back up this folder off site, but that's a feature i'll implement soontm.
<br/>
<br/>
You can find additional information [here](https://github.com/dani-garcia/vaultwarden)

## Omnitools
This is a very simple site that has a bunch of tools for, pdfs, images, videos, gifs and so on. Not very useful, until you need it, then it's the best thing to ever be concieved.

## Jellyfin
Jellyfin is a media streaming service for movies, tv shows and/or music. 
Important to note that this docker-compose file is set up for **Nvidia** gpus, used for hardware transcoding.
You can find more information in the docs [here](https://jellyfin.org/docs/)

# How I connect to my homelab
Here's a quick overview of the system I use to actually talk to my homelab. I use:
- **Tailscale**
- **Pihole** (more on that can be read on the RaspPi section)
- **Nginx** (*not* Nginx Proxy Manager).

### Flow of data
I use a set of 2 domains for each of my services, one ending in `.lan`, the other ending in `.tail`. 
<br/>
`.lan` is connected, obviously, via a lan IP (192.168.\*.\*), and `.tail` is connected via an IP provided by tailscale (100.\*.\*.\*).
<br/>
These domains are *not* public. They are custom domains defined as local DNS records in Pihole. 
As they're not public, but I still want https, I use a self signed SSL certificate, which I generated via this command:

```
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout malzalab.key -out malzalab.crt \
  -subj "/CN=malzalab.lan" \
  -addext "subjectAltName=DNS:*.malzalab.lan,DNS:*.malzalab.tail"
```
With this command, you'll get `malzalab.crt` and `malzalab.key`, which you use in Nginx, and you have to install the .crt file into either your browser, or your OS trust store.
<br/>
While on the topic of Nginx, it's used to get the port on which my services run.
I use raw Nginx config files, and I used [these docs](https://nginx.org/en/docs/) to help me set up nginx.
<br/>

# Backup
There are lots and lots of ways to do backups. I chose the more barebones way using **rsync** and a simple shell script:
```
#!/bin/bash

mkdir -p /path/to/backups/log
LOG_FILE="/path/to/backups/log/nightly_backup_$(date +%F).log"

{
  echo "==========IMMICH BACKUP=========="
  rsync -avh --delete \
    --exclude='encoded-video' \
    --exclude='thumbs' \
    /path/to/immich-data/ /path/to/backups/immich-data/

  echo "==========VAULTWARDEN BACKUP=========="
  rsync -avh --delete /path/to/vaultwarden /path/to/backups/vaultwarden
} > "$LOG_FILE" 2>&1

```

Since currently I'm limited on storage space, I back up my most important stuff in images and passwords.
This is a very temporary and bad solution, since here I'm just copying stuff from my prone-to-failure hard drives to my boot SSD. I will add a proper 3-2-1 solution soon(tm)

