---
layout: post
title:  "Installing Proxmox and setting up Windows 11 VM"
date:   2026-01-06 23:00:43 +0100
categories: how-to proxmox
tags: ["virtualization", "proxmox", "vm"]
---

# Installing Proxmox and setting up a Windows 11 VM

This is a digest on all the steps I took to install Proxmox in my new workstation and set up a Windows 11 VM.


## Installing Proxmox VE

* Prepare executable USB and adjust BIOS boot sequence of the host machine
* Install using GUI, but change the boot command before - simply delete everything after ```quiet/splash``` and replace with:

```
nomodeset nouveau.blacklist=1 modprobe.blacklist=nouveau i915.modeset=0 intel_iommu=off

```

## Post-Install Tasks

* Create additional user to connect to the node (```adduser```), give sudo permissions to this newly created user.
* Adjust sshd_config to disable root login, accept only public-key login.

## Create VM with Windows 11

* It's possible to export/import an activation key, provided it's a RETAIL type key (not OEM).
* To figure out the type of key, use ```slmgr /dli```

***The following steps work only on RETAIL licenses***, which are associated to a Microsoft account. Other license types are linked to hardware and it's not possible to transfer them to another machine (no matter if physical or virtual).

* Create system image of the physical machine with Windows 11.
* Save activation key and unload it from the physical machine with ```slmgr /upk```
* Download Windows 11 .ISO from [Microsoft's Official Download Site](https://www.microsoft.com/de-de/software-download/windows11) and the .ISO with VirtIO's drivers ([link](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/?C=M;O=D)). These drivers are needed to avoid hardware compatibility issues on the VM.
* Upload both ISOs to Proxmox Server and create a new VM with Windows 11 with the following specs:

### VM Specifications

* ISOs: Windows 11 and VirtIO
* Graphic card: Default, type q35
* BIOS: OVMF (UEFI), add TPM Module, SCSI Controller: VirtIO
* CPU: 4 cores
* RAM: min. 4 GB, activate ballooning, 2 GB increment
* HDD 100 GB
* Network card model: VirtIO paravirtualized

* Change the boot sequence (first should be ISO with Windows 11) and power on the VM

* To install the operating system, just follow [Decatec's Tutorial](https://decatec.de/home-server/proxmox-ve-windows-11-als-virtuelle-maschine-installieren-und-einrichten/)

## Installing XRDP

* Once Windows 11 is installed, we can use RDP to connect to a Linux machine. In this case, the destination of this RDP connection is a Debian with KDE Plasma.
* On the destination machine, install XRDP, allow autostart and check that it's started

```
sudo apt install xrdp
sudo systemctl is-enabled xrdp
sudo systemctl status xrdp
```

* Create certificate and save in ```/etc/xrdp/certs``` (create directory if it doesn't exist yet), change ownership: ```sudo chown -R xrdp:xrdp /etc/xrdp/certs```
* Adjust permissions for the certificate and the private key

```
sudo chmod 0644 /etc/xrdp/certs/certificate.pem
sudo chmod 0600 /etc/xrdp/certs/privatekey.pem
```

* Adjust parameters in ```/etc/xrdp/xrdp.ini```:

```
security_layer=tls
certificate=/path/to/certificate.pem
key_file=/path/to/privatekey.pem
ssl_protocols=TLSv1.2, TLSv1.3
```

* Create file ***~/.xsession*** with the following contents:

```
!/bin/sh
export XDG_SESSION_TYPE=x11
export KDE_FULL_SESSION=true
unset WAYLAND_DISPLAY
unset DBUS_SESSION_BUS_ADDRESS
exec dbus-run-session startplasma-x11
```

* Make .xsession executable: ```sudo chmod +x ~/.xsession```
* Backup file ***/etc/xrdp/startwm.sh*** and change the contents of the original file as follows:

```
#!/bin/sh
if [ -r ~/.xsession ]; then
  exec ~/.xsession
else
  exec startplasma-x11
fi

```

* Restart XRDP: ```sudo systemctl restart xrdp```, start RDP session from the Windows machine
* In case you experience connection problems, check contents of ***~/.xsession-errors*** in the destination Linux machine

