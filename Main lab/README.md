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
  - Immich
  - Vaultwarden
  - Omnitools
  - Jellyfin
  - Nextcloud
  - Metube
  - Arr* stack
- Nginx reverse proxy
  
## Docker
Most of the stuff I actually use (with a few exceptions like Pi-hole) run via docker and docker compose. If you've never used docker, there are plenty of tutorials online, but to use my config, 
all you have to do is install docker from [here](https://docs.docker.com/engine/install/), 
and either copy a `docker run` command, or navigate to a folder with a `docker-compose.yml` file and type in `docker compose up -d`.
For more information, use the interwebs or [check the docs](https://docs.docker.com/).

## Portainer
Portainer is a docker management tool where you can manage images, stacks, containers and other stuff. To install it, I followed the [portainer docs](https://docs.portainer.io/start/install-ce/server/docker/linux), but if you're lazy, 
basically run `docker volume create portainer_data` to create a volume needed by docker, then run `docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts`. 
After that, access the webui at https://localhost:9443.

## Watchtower
Watchtower is a tool that automatically scans and updates docker images for all of your current active docker containers.
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
<br/>
For more info about all of this, check out the [docs](https://immich.app/docs/overview/welcome/)
