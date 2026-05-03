---
layout: post
title:  "Remediating Copy Fail Vulnerability"
date:   2026-05-03 18:29:32 +0200
categories: how-to security
tags: ["security", "debian"]
---

Kudos go the user gapreg, who posted on [Reddit](https://www.reddit.com/r/selfhosted/comments/1szq9e5/comment/oj5m1em/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) how to remediate it by disabling the module and stopping it from loading dynamically.

A workaround to patch your systems while an official kernel update coming from your distro becomes available (afaik Debian already has one).

* Check if module exists: ```modinfo algif_aead```
* Check if it's currently loaded: ```lsmod | grep algif```
* Check if it can be dynamically loaded: ```cat /proc/sys/kernel/modules_disabled```
* Stop from loading it dynamically: ```echo "install algif_aead /bin/false" > /etc/modprobe.d/disable-algif-aead.conf```
* Remove from memory if loaded: ```rmmod algif_aead 2>/dev/null```

