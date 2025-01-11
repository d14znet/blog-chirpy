---
layout: post
title:  "Resetting a Raspberry pi"
date:   2025-01-11 18:55:00 +0200
categories: how to backup raspberry pi
tags: ["how to", "backup", "raspberry pi"]
---

**Happy New Year!**

During the last months I didn't have time to start any new projects, since I was having trouble with my Raspberry Pi.

Neither logs nor monitoring wouldn't give me any clue, there were also no signs of overheating or sudden power loss. 


# The Problem

My Raspberry Pi runs a couple of containers, one of them is PiHole. If it loses connectivity from the local network, my hosts are practically isolated from the Internet, since they cannot resolve domain names into IPs.

Before setting up a backup PiHole I wanted to know what was going on with my Pi. Turned out to be more difficult than I imagined.

**There was nothing wrong!** Syslog wouldn't show anything suspicious. Neither did dmesg. The only thing I found related to any network connectivity problems were messages from the Avahi Daemon - the Pi losing its addresses and getting them back some seconds later.

My router and DHCP server didn't offer me any clue either. The Pi's temperature was normal. I assumed something was wrong with its networking because I couldn't ping the Pi's address, neither could I SSH to it to have a look at the running processes, services and so on. Powering it off and then on again was the only solution, which turned to be pretty annoying in almost no time.


# The Solution

I searched the web for similar problems and its troubleshooting and tried to narrow down the results for something 'recent'. And so I stumbled upon [this thread on Raspberry Pi Stack Exchange](https://raspberrypi.stackexchange.com/questions/143763/raspberry-pi-periodically-crashes-and-breaks-my-access-point-until-i-reboot-the).

I decided to backup my working directories and critical files on the Pi, flash the microSD card again and restore the backup. **Here we go**.

- Make any necessary changes to your network (announce a different DNS server on your DHCP configuration, for example) and stop services on your Pi
- Make a tarball to backup files and directories: ```tar -cvf <name-of-tarball>.tar /path/```
- Remember to include on this tarball important configuration files (such as SSH), you can also get a list of installed packages with ```apt list --installed```
- Shutdown the Pi, extract the microSD card and flash it with your preferred [OS image](https://www.raspberrypi.com/software/operating-systems/). For this you can use the [Pi Imager](https://www.raspberrypi.com/software/) or [Balena Etcher](https://etcher.balena.io/) (my choice).
- Insert the microSD card, perform the first configuration of the OS (hostname, network, software packages...)
- Transfer the backup tarballs and untar them accordingly: ```tar -xvf <name-of-tarball>.tar -C /path/``` - note that you only need to specify a target path with -C if you want to untar the files on a different directory than the working one.
- Download and update any other software packages that might be missing. For Docker, I followed the [official documentation for Debian](https://docs.docker.com/engine/install/debian/).
- At this point, you should have your Pi up and running with all backups deployed. Get your containers running again, and update/upgrade any packages if you haven't yet.


# The Outcome

I'm looking forward to see if this really solves the problem! I'll have an eye on it for the next couple of weeks and then update this post with the results.