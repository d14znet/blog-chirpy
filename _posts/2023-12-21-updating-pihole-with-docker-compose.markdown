---
layout: post
title:  "Updating PiHole with Docker-Compose"
date:   2023-12-21 19:52:43 +0100
categories: how-to pihole docker
tags: ["raspberry pi", "how to", "docker", "pihole"]
---

# Updating PiHole with `docker-compose`

## Disclaimer & Credits

> These are the instructions I follow whenever there is an update available for the PiHole Docker image.
> They are based on the steps described on the [Official Repository](https://github.com/pi-hole/docker-pi-hole#upgrading--reconfiguring).


## Instructions

* List downloaded images - this will get us the Image ID of the last downloaded PiHole image (which is now outdated): `sudo docker images`.
* Delete outdated image with `sudo docker image rm ***<ID of image>***`
* Check that `docker-compose` will use the latest image version: `sudo nano docker-compose.yml` - I use `pihole/pihole:latest` on the directive under **services > pihole > image**.
* Download latest image version with `sudo docker pull pihole/pihole:latest`.
* Take down container - **WARNING**: Clients using the PiHole as DNS Server won't be able to resolve addresses at this point and until you take the container up again. If your PiHole uses its own IP address as DNS-Server, be sure to change it on `/etc/resolv.conf`. You can change it to your router IP or to a [public resolver](https://en.wikipedia.org/wiki/Public_recursive_name_server), for instance.

```console
cd /pihole/directory
sudo docker-compose down
```

* Renew docker-compose images (requires DNS, see above): `sudo docker-compose pull`
* Take container up again: `sudo docker-compose up -d`


## Last steps 
If you changed the DNS server on your `/etc/resolv.conf`, remember to change it back to your PiHole's IP address.
