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
  - Portainer
  - Watchtower
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
WIP
