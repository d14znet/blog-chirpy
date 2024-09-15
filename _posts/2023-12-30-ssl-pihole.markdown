---
layout: post
title:  "Configuring self-signed SSL on PiHole's Web Interface"
date:   2023-12-30 00:29:43 +0100
categories: how-to pihole docker
tags: ["raspberry pi", "how to", "docker", "pihole", "certificates"]
---

# Configuring self-signed SSL on PiHole's Web Interface

I use PiHole with Docker - I'm aware that the best Docker approach to allow SSL would be setting up a proxy.
However, I wanted to test how to do it using the container that was already running, without setting anything else up.
This also helped me refreshing the theory about how to allow SSL on a webserver: generate certificates, tweak up the config, force HTTPS over HTTP, and so on.
Also, I'm not quite sure if my PiHole setup will stay as it is now or if would move it to a VM. Anyway, it was a good excuse to get my hands on Docker Compose, Lighttpd and OpenSSL.


## Disclaimer & Credits

As I can't afford an own domain name by now, these instructions are aimed for a **self-signed certificate**.
Obviously it's not as trustworthy as a CA-signed one, and most browsers will complain against it, but that's how it is right now.
As soon as I can afford my own domain, I'll be most happy to create a new How-To guide with Certbot.

I already worked on a little project to automatize certificate issuing and renewal for Linux-based webservers using Certbot, so I'm looking forward to it.

> For the most part I followed these instructions:
>   - WaLLy3K - [How can I enable HTTPS (SSL/TLS) for my Pi-hole Web Interface?](https://discourse.pi-hole.net/t/enabling-https-for-your-pi-hole-web-interface/5771)
>   - Brandon Lee - [Configure Pi-hole SSL using a self-signed certificate](https://www.virtualizationhowto.com/2021/12/configure-pi-hole-ssl-using-a-self-signed-certificate/)
> While troubleshooting problems with binded mounts, Docker's official documentation about [bind mounts](https://docs.docker.com/storage/bind-mounts/#mount-into-a-non-empty-directory-on-the-container) helped a lot.


## Instructions

### Generate self-signed certificate with OpenSSL

You can do this on your Docker container or directly on the host - if you decide for the latter, keep in mind the files' permissions (as they need to be read by users on the Docker container).

```console
openssl req \
       -newkey rsa:2048 -nodes -keyout pihole.key \
       -x509 -days 365 -out pihole.crt
```

OpenSSL will generate and output the keys on the current directory.
After this, we need to combine both generated files (.key and .crt) into a single .pem file: ```cat pihole.key pihole.crt > pihole.pem```

### Set up Lighttpd's configuration to allow SSL

Once we've generated the keys, we need to tell Lighttpd to load SSL, where to find the keys for the current webserver, and also to force HTTPS instead of plain-text HTTP. 

For this, create a file like ```20-pihole-ssl.conf``` and paste the following:

```console
#Load OpenSSL
server.modules += ( "mod_openssl" )

#Specify which port should be used for SSL, and where the certificate is located
setenv.add-environment = ("fqdn" => "true")
$SERVER["socket"] == ":443" {
        ssl.engine  = "enable"
        ssl.pemfile = "/path/to/cert.pem"
        ssl.openssl.ssl-conf-cmd = ("MinProtocol" => "TLSv1.3", "Options" => "-ServerPreference")
}

# Redirect HTTP to HTTPS
$HTTP["scheme"] == "http" {
    $HTTP["host"] =~ ".*" {
        url.redirect = (".*" => "https://%0$0")
    }
}
```

Store the `.conf` file under /etc/lighttpd/conf-enabled/. In case your setup has a system user for Lighttpd (like `www-data`, make sure the permissions are set up appropriately for this config file and for the `.pem` file.

### Restart Lighttpd

Depending on your setup, you might restart running ```sudo service lighttpd restart``` (that's what I did) or ```sudo systemctl restart lighttpd```.

Now when accessing PiHole's web interface, you will see a warning from your browser - since the self-signed certificate isn't trustworthy enough. Proceed anyway and you'll see the login site of PiHole uder https.


## Combine with `docker-compose`

### Problem + Solution 

So far the process is quite straightforward. The real deal was combining this with a `docker-compose` setup. As far as I knew, anytime I'd have to take down PiHole (because of an update, for instance), the **whole SSL configuration will be reset** and I would have to start from the beginning. Quite annoying.

The docker-compose file I use already bind-mounts a couple of directories, so that all of their contents are backed up and automatically restored whenever I need to restart the PiHole. I thought this might work for Lighttpd's SSL configuration as  well, but I encountered some problems on the way:

  * A bind mount of `/etc/lighttpd` to the Docker host didn't work. Refer to [Mount into a non-empty directory on the container](https://docs.docker.com/storage/bind-mounts/#mount-into-a-non-empty-directory-on-the-container) for further explanation. Using my previous docker-compose file, lighttpd will be automatically started on the PiHole container, enabling the server configuration:
    * Bind mounting an empty `etc-lighttpd` to PiHole's `/etc/lighttpd` **would render the web interface useless**. The source's empty contents would propagate to the container, erasing all configuration files under conf-available and conf-enabled - which are needed by lighttpd to service the web interface.
	* Bind mounting empty `etc-lighttpd` and child directories `conf-available` and `conf-enabled` didn't work either. It seems that using the `volumes` directive on Compose without any further configuration creates a bind mount with read/write permissions by default. This didn't work as I expected for the approach I had in mind.
  * At this point I had spent much more time than I really had and decided to go for a workaround, which is described below. 
  
Basically I thought it would be possible to let PiHole start Lighttpd and generate all configuration files. Those would be then written into the Docker host, creating a sort of **backup of Lighttpd's configuration** (where I could store the .pem too). This backup will persist on the Docker host, so that the SSL configuration will also persist after taking the container down and up again. I decided to stick to the bind mount approach, but giving it a twist.
 
### Bind mount for config file and certs

* Create a directory on the Docker host containing `ssl` subdirectory (containing the .pem file) and the config-file mentioned before.
* Make sure the directories and their files have the appropriate permissions. If needed, set them up correctly with `chown` / `chmod`.
* Take your container down. On the docker-compose file, create a new bind mount point between the new directory and the container's filesystem:

```yaml
#[...]
services:
  pihole:
    #[...]
    volumes:
      - './your/local/path:/path/in/container'
```

### Soft-link the configuration and restart Lighttpd service

* Take the container up again and open a terminal to it with `docker exec -it <container-name> bash`.
* Navigate to the path you just bind mounted. You will find there the config file and the ssl folder.
* Soft-link the config file to /etc/lighttpd/config-enabled: `ln -s /path/in/container/20-pihole-ssl.conf /etc/lighttpd/conf-enabled/20-pihole-ssl.conf`
* Restart lighttpd with `service lighttpd restart`


## Retrospective & To-Dos

This solution isn't perfect and there are some points that would still require manual configuration whenever the PiHole is restarted. On a brighter side, this manual configuration is so minimal that it can be easily automatized (I think!).

I'm pretty sure there is a way to populate Lighttpd's configuration into the Docker Host as I originally thought - for this I'll need to dive deep into Docker volumes and bind mounts' propagation settings. But exams are coming soon and I don't know how much spare time I'll have in the upcoming months (probably none).

All this leaves me with the following **To-Dos**, ordered by feasibility:

* Automatize soft-linking of lighttpd-configuration and service restart.
* Tweaking the container's volumes and/or bind mount propagation settings to make it work as I originally thought.
* Ignoring all of this and going the best-practice way - placing a proxy server before the container to handle all SSL traffic.
