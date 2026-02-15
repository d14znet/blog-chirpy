---
layout: post
title:  "Setting up a private Git Server"
date:   2026-02-15 15:01:32 +0100
categories: how-to git proxmox
tags: ["git", "proxmox"]
---

# Setting Up a private Git Server

I'm trying to keep up as fast as possible with all the knowledge I feel I've missing out. I already use GitHub and GitHub pages to host this blog and other personal repos, however I would like to explore how would it be to have my own personal Git Server on my Proxmox VE.

## Prepare VM and Install GitLab

* I adapted my Debian 13 cloud-init image to 4 GB RAM and 30 GB disk space. Everything else (users, SSH-keys, etc) stayed the same.
* Power on the machine, install updates and all necessary packages
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y tzdata perl
sudo apt install -y postfix
```
* Download GitLab installation script and start installation providing either the hostname or the IP of the server where Git should run. 
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://<server-fqdn or server-ip>" apt install gitlab-ce
```
> Installation might take up to 10 minutes.
{: .prompt-info }

## Setting Up GitLab Instance and Test

* On a browser, call ```http://<server-fqdn or server-ip>```, you'll be prompted for login on GitLab's Web UI.
* Don't forget to change root's initial password using [GitLab's Instructions](https://docs.gitlab.com/security/reset_user_password/#reset-the-root-password)
* Create a Group, create a Project and assign it to the group.

![image](assets/img/gitlab-create-project.png)

* Create a couple of users, add them to the group with the **Developer** role and set SSH keys so that key can commit and push their content to the project folder.

**Cloning repo and committing changes**

* From a remote machine, try cloning the repo you just set up: ```git clone git@<server-IP>:<group-name>/<project-name>.git```. It should clone the structure of the project you just set up, including the automatically generated ```README.md```.
* Also from the remote machine, create a branch, add some changes and try committing / pushing them:

```
cd gitlab/<project-name>
# add folders, files, etc
git checkout -b feature-test
git add .
git commit -m "First commit"
git push origin feature-test
```

* Once logged in into GitLab Web UI, you will be prompted about a **merge request**. Change to your root user to approve the merge

![image](assets/img/gitlab-merge-request.png)

![image](assets/img/gitlab-approve-merge.png)

> Our test user has a Developer Role, which prevents them from writing changes directly to the main branch. If you wish to skip this workflow entirely, just change the user role to **Maintainer**, or change the Developer role to allow those users to push changes to main.
{: .prompt-info }

**Setting Up SSL and NGINX redirect**

* Issue SSL certificates
* Create folder structure on GitLab server and save certificates there

```
sudo mkdir -p /etc/gitlab/ssl
sudo chmod 755 /etc/gitlab/ssl
```

* Adjust contents of ```/etc/gitlab/gitlab.rb``` and reconfigure instance to apply changes with ```sudo gitlab-ctl reconfigure```

```
external_url 'https://<server-fqdn>'
[...]
letsencrypt['enable'] = false
[...]
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/<server-fqdn>.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/<server-fqdn>.key"
nginx['ssl_protocols'] = "TLSv1.2 TLSv1.3"
nginx['listen_port'] = 443
nginx['listen_https'] = true
```

