---
layout: post
title:  "Setting up SSH Server on Windows 11"
date:   2026-02-08 19:06:43 +0100
categories: how-to windows
tags: ["sysadmin", "windows"]
---

# Setting up SSH Server on Windows 11

Just writing down all the steps in case I have to set this up again... which honestly I hope not to. Everytime I have to set up something on Windows I'm reminded of why I stick to Linux.

## Install OpenSSH Server

* Look for "Features" on the search bar and click on Optional Features > See Features > Show available features > search for **OpenSSH Server** and install it.
* Open Services and change startup type to Automatic for both OpenSSH Authentication Agent and OpenSSH Server. Start both services.
* Check status of installation (PowerShell as Administrator) ```Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Server*'```
* Allow incoming SSH connections with ```New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22```

## Tweak up configuration

* To allow public key authentication only, first create an **authorized_keys** file on the .ssh folder of your user.

> Watch out for correct permissions! This file should be accessible by SYSTEM and your user only. Click on Properties > Security > Advanced and Disable inheritance (Convert inherited permissions into explicit permissions on this object). SYSTEM and your user should have full control over the file.
{: .prompt-info }

* Open Program Data (click "Run" on the search bar and run the command ```%programdata%```) and adjust the contents of ssh > **sshd_config** (**NOTE**: you might have to log in as Administrator or give permissions to your user in order to be able to edit this file)
  * No password authentication allowed
  * Only public key authentication
  * Comment out the last 2 lines of this config file

![image](/blog-chirpy/assets/img/sshd_windows.png)

* Restart OpenSSH-Server and OpenSSH Authentication Agent after the changes

