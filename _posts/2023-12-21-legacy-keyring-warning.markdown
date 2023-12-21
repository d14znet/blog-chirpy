---
layout: post
title:  "How to fix 'Legacy Keyring' Warning while updating Raspberry Pi"
date:   2023-12-21 20:19:43 +0100
categories: how-to raspberry-pi
tags: ["raspberry pi", "how to", "legacy keyring"]
---

# How to fix Legacy Keyring Warning while updating Raspberry Pi

## Disclaimer & Credits

> I followed the instructions from [Fixing "Key is stored in legacy trusted.gpg keyring" Issue in Ubuntu](https://itsfoss.com/key-is-stored-in-legacy-trusted-gpg/)


## Warning Message

Since a couple of weeks I was getting a warning when updating my Raspberry Pi:

> Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.

Although it was only a warning and updates were running without problems, this is what I did to get rid of the issue:


## Instructions

1. Get a list of keys with `sudo apt-key list`.
2. The key that is causing problems is the one stored under `/etc/apt/trusted.gpg`.
3. Once you have found the key, copy the last 8 digits of the `pub` block, without spaces.
4. Export the deprecated key with apt-key and import it using GPG:

![We need to import the last 8 digits of the key listed under /etc/apt/trusted.gpg](/assets/img/deprecated-key-raspi.png)

``` console
sudo apt-key export 90FDDD2E | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/***<repo-name>***.gpg
```


## Last steps

* You can list the contents of `/etc/apt/trusted.gpg.d/` to check that the import was successful.
* Now when you update the repositories the warning message won't be displayed anymore.

