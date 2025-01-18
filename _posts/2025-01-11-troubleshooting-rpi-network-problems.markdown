---
layout: post
title:  "Troubleshooting Raspberry Pi Network Problems"
date:   2025-01-11 18:55:00 +0200
categories: how to troubleshooting raspberry pi
tags: ["how to", "troubleshooting", "raspberry pi", "network", "firmware"]
---

**Happy New Year!**

During the last months I didn't have time to start any new projects, since I was having trouble with my Raspberry Pi.

Neither logs nor monitoring wouldn't give me any clue, there were also no signs of overheating or sudden power loss. 


## The Problem

My Raspberry Pi runs a couple of containers, one of them is PiHole. If it loses connectivity from the local network, my hosts are practically isolated from the Internet, since they cannot resolve domain names into IPs.

Before setting up a backup PiHole I wanted to know what was going on with my Pi. Turned out to be more difficult than I imagined.

**There was nothing wrong!** Syslog wouldn't show anything suspicious. Neither did dmesg. The only thing I found related to any network connectivity problems were messages from the Avahi Daemon - the Pi losing its addresses and getting them back some seconds later.

My router and DHCP server didn't offer me any clue either. The Pi's temperature was normal. I assumed something was wrong with its networking because I couldn't ping the Pi's address, neither could I SSH to it to have a look at the running processes, services and so on. Powering it off and then on again was the only solution, which turned to be pretty annoying in almost no time.


## The Solution

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


## The Outcome

I'm looking forward to see if this really solves the problem! I'll have an eye on it for the next couple of weeks and then update this post with the results.

### Update 1

Oh boy :( not even 24 hours have passed and my Pi was again offline. I was so annoyed that I started collecting information and evidence from everywhere I could:

- The network crash happened around the same time my router started reconfiguring its WLAN channels.
- Somewhere during my prior search I read something about country codes for WLAN channels. But I already had configured this with raspi-config after flashing the new OS.
- Looking at journalctl entries around the time of WLAN channel reconfiguration I found something interesting:

![brcmfmac kernel messages](/assets/img/brcm.png)

**brcmfmac was failing to set a proper channel and was causing a ton of kernel messages!** Finally something I could start working with! Of course others were [reporting the same issue on GitHub](https://github.com/raspberrypi/linux/issues/6049#issuecomment-2485431104). I followed the suggested instructions to disable the ```DUMP_OBSS``` feature from the Broadcom kernel module.

- Performing a ```cat /sys/kernel/debug/ieee80211/phy0/features``` will show that DUMP_OBSS is activated.
- You can now try removing the kernel module with ```sudo modeprobe -r brcmfmac```. Honestly this didn't work for me as the module was always on use - no matter if I had disabled the service wpa_supplicant first. ```sudo modprobe -r brcmfmac-wcc brcmfmac``` did the trick and I was able to remove the kernel module.
- Use ```sudo modprobe brcmfmac feature_disable=0x200000``` to disable DUMP_OSS and restart the service wpa_supplicant.
- Check again ```cat /sys/kernel/debug/ieee80211/phy0/features``` to see if the feature has been disabled - note that ```phy0``` might have turned to ```phy1``` if you had to force removal.
- If this worked, create a file on /etc/modprobe.d/brcmfmac.conf and add there the following, so that the firmware always starts with the feature disabled: ```options brcmfmac feature_disable=0x200000```.
- When running again, DUMP_OBSS should be out of the list.

~~I'm keeping my fingers crossed that this will finally solve the problem. Otherwise, I'll keep updating this post :D~~

### Update 2

Not even 4 hours after this fix the wlan0 interface was again not responding. Since this was leaving all clients on my network without Internet access, I focused on a **quick workaround** - connect the Pi via Ethernet to my local network and swap IP-addresses between the eth and wlan interfaces, so that clients will seamlessly reach their DNS server. 

I restarted again the Pi and looked every other day at dmesg and journalctl for any crashes or errors. A couple of days ago I could only find an error message related to an unknown frame, so that I repeated the steps to disable DUMP_OBSS and tried adding a ```options brcmfmac feature_disable=0x82000``` to the kernel module, which didn't solve the problem. You can find more information on this configuration flag on [BCM4345 (Rpi 3+,Rpi 4 ) - CTRL-EVENT-ASSOC-REJECT on WPA2 AP when using SWSUP and brcmfmac](https://github.com/RPi-Distro/firmware-nonfree/issues/34#issuecomment-1378760628). It basically disables SAE, which was recommended on the thread for those having problems with authentication, that wasn't my case.

Since then the wlan interface hasn't stopped working! Anyway I think I will work on a way to have all logs more accessible, so that I can check if the problem is solved once and for all. But this seems a topic for another blog post :)
