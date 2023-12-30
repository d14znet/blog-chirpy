---
layout: post
title:  "Initializing new local repo and cloning from remote source"
date:   2023-12-30 14:29:43 +0100
categories: how-to github
tags: ["how to", "github"]
---

# Initializing new local repo and cloning from remote source

I already have a remote repository on GitHub, now I'd like to work on it also from my local Windows machine.

> Make sure you have [Git](https://git-scm.com/) installed!
{: .prompt-info }

## Set up and initialize local repository

* Navigate to the path you'd like to create the local repository and create a folder for it: 

```console
cd C:\Users\user\github
mkdir .\localrepo
cd .\localrepo
```
* Initialize your new local repository with `git init`.

## Create SSH keys for your local machine

* Open the terminal and **start the SSH Agent** on the background: `ssh-agent s`
* Create a **new public-private key pair** for your local machine: `ssh-keygen -t rsa -b 4096`
  * By default, `ssh-keygen` will store your keys under `C:\Users\user\.ssh\`. During prompt, you can specify another path though.
  * It is recommended to protect your keys with a **passphrase**!
* Go to `GitHub > Settings > SSH and GPG Keys` and create a **New SSH Key**.
* Output your public key with `cat C:\path\to\keys\pubkey` and **paste the content** into GitHub. Save your new key.
* On the terminal, **load your new public key** into the SSH Agent: `ssh-add C:\path\to\keys\pubkey`

## Clone remote repository via SSH

* On GitHub, open your repository and go to `Code > Clone > SSH`. **Copy the snippet into your clipboard**:

![Copy the code snippet to clone your repository via SSH](assets/img/ssh-clone-repo.png)

* On your local terminal, navigate to the local repository and run `git clone git@github.com:your-github-user/your-remote-repo.git`.
* GitHub will check if the provided public key is authorized to use the remote repository. The contents of your remote repository will start propagating into your local repository.

## [Optional] Make a test commit from your local repository

To make sure everything went right and synchronization works as expected, we can create a new file in the local repository and then commit it to the remote one.

* [Also optional] On your local repository, open a terminal and run `git pull`. As we didn't change anything so far, the output will tell us that *everything is already up-to-date*.
* Create a new file or modify an existing one. Save your changes and add the file to Git: `git add /path/to/file`.
* Create a commit: `git commit -m "Test commit"`
* If you didn't set up a user name and e-mail yet, Git won't commit the changes. The output on terminal is pretty self-explanatory on what you need to do - simply follow the instructions and commit the changes again:

![Follow the instructions on the terminal output to set a user name and e-mail for the repository](assets/img/setup-git-config.png)

* Finally, push the changes to the remote repository with `git push`.

