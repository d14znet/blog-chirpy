---
layout: post
title:  "Restoring Raspberry Pi after crash"
date:   2026-01-19 20:19:43 +0100
categories: how-to raspberry-pi
tags: ["raspberry pi", "nextcloud", "pihole", "recovery"]
---

# Restoring Raspberry Pi after crash

The crash was caused by an OpenMediaVault installation via bash script.
OpenMediaVault changes the *whole* network config of the Raspberry Pi - it even uninstalls/activates other network and systemd services for network administration. OVM changed so many parameters that a simple uninstall didn't help - the Raspberry Pi was rendered completely useless.
That's why I decided to reflash the microSD card and tried to recover as much information as possible (Github repositories, containers, etc).

## Reflash and data recovery

* Connect microSD to another machine and mount rootfs - then copy contents of rootfs with ```sudo rsync --progress <source> <dest>```
* Reflash microSD with [Raspberry Pi's official imager](https://www.raspberrypi.com/software/). This allows us to create a system image with specific parameters: WLAN SSID, username and password, SSH public key, etc.
* Once flashed, connect microSD to Raspberry Pi and start system.
* If a SSH server was installed during flashing, copy all files we want to recover using ```scp```. Add directories first to a tarball if you also want to recover them.


## Reinstall software

* Reinstall all necessary software, such as docker: ```sudo apt install docker.io```


## Restart containers

* *PiHole*: For this one I just had to recover the directory containing the docker-compose and start the container as usual: ```sudo docker-compose up -d```. New images will download automatically. Once the container is up and running again, check DNS connectivity and adjust contents of ```/etc/resolv.conf```.

* *Nextcloud*: For this I had to *adjust permissions* and start all sub-containers in a specific order:

1. The user/group *www-data* need access to the volumes we defined on the docker-compose for the Nextcloud service. Similarly, user/group *UID/GID 999* need permissions for the database volumes.
2. If we start docker-compose as we usually do, Nextcloud will initialize before the database can read all recovered data. Instead, we have to start the database first with ```sudo docker-compose up -d mariadb```. Then we check if the database is up and running and accepting incoming connections: ```sudo docker logs -f mariadb```. 
3. Connect to the database and check if Nextcloud's latest session tables have been recovered:

```
sudo docker exec -it mysql -it mariadb mysql -u'db_user' -p'password' db_name #(adjust your connection parameters)
SHOW TABLES;
```

4. Once we made sure that the database is accepting incoming connections and contains data from the last session, we repeat the steps for all other services; first Redis ```sudo docker-compose up -d redis``` (check status with ```sudo docker logs -f redis```); and then Nextcloud ```sudo docker-compose up -d nextcloud```. Same as before, we check on Docker's logs that Nextcloud is up and running and Apache is ready for incoming connections.
5. It's possible that we need to install an update once we access Nextcloud's URL. In this case, we can install it within a bash session in the container:

```
sudo docker exec -it nextcloud bash
./occ upgrade

```

