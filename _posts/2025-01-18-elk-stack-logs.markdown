---
layout: post
title:  "Installing ELK Stack to collect logs"
date:   2025-01-18 20:37:00 +0200
categories: how to monitoring raspberry pi
tags: ["how to", "monitoring", "raspberry pi", "elk"]
---

As I mentioned on my previous post, I've been spending much time lately looking for errors and information on my Raspberry Pi's logs. **Not that it bothers me!** - a considerable part of my work consists of reading logs, after all. I was only wondering if there was a way to collect the logs in a way that makes them easier to navigate, filter out for relevant info, and so on.

I already had contact with Prometheus and Grafana and knew about the existence of Loki - which basically collects your logs in the same way as Node Exporter collects metrics for Prometheus. But for this experiment I wanted to give a try to the **ELK-stack** (**E**lasticsearch, **L**ogstash and **K**ibana). During the final project for my vocational training, I discarded the ELK stack precisely because I wasn't interested on collecting log information. This time it's the other way around and I'm looking forward to see if it meets my expectatives and as always, to learn something new along the way!


## Installation

### Elasticsearch

The [official website](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html) has loads of documentation and alternative ways to get the stack up and running. They also provide a script to run the whole stack as a container. For my case I'll stick to **apt** (I wrote down the instructions for deb-package in case I need to start again from scratch).

#### Using APT
- Copy Elasticsearch's PGP Key: ```wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg```
- Add their repositories with: ```echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list```
- Update your package manager and install Elasticsearch: ```sudo apt update && sudo apt install elasticsearch```

This will install the latest available version for Elasticsearch. Alternatively, it is possible to download the deb-package from their website.

#### Using deb-package
- Check your CPU architecture with ```uname -a```.
- Check the [official download](https://www.elastic.co/downloads/elasticsearch) site to get the link for your architecture, and download the package: ```sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.0-arm64.deb```
- Install the package: ```sudo dpkg -i elasticsearch-8.17.0-arm64.deb```

> During installation, a password for a super-user will be automatically generated, don't forget to save it!
{: .prompt-info }


After download, you can enable and start the service with systemd:

```console
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

And also test if the service is running correctly - Elasticsearch runs on port 9200, so we can send a HTTP request using the certificate that has been automatically generated during installation. For the following command to work, you might want to save your superuser password as a Shell variable with ```export $ELASTIC_PASSWORD="your_pw"```:

```console
sudo curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200 
```

### Logstash and Kibana

- If you downloaded Elasticsearch using the APT-method, you already have access to Elastic's repositories and can download Logstash and Kibana by simply running an ```sudo apt install```. If you downloaded the deb-packages for Elasticsearch, you can do exactly the same for Logstash and Kibana - here you can find download links for each package according to your CPU architecture: [Logstash](https://www.elastic.co/downloads/logstash), [Kibana](https://www.elastic.co/downloads/kibana).
- Same as before, reload systemd daemons, enable and start the services we just installed.


## Getting Started

First we need to properly configure Kibana and **connect it to our Elasticsearch instance**.

- Check Kibana's configuration file on ```/etc/kibana/kibana.yml``` and change according to your needs - you might want to use an alternative port number or another binding address. Remember to reload Kibana if you change anything on this file.
- Connect Kibana to Elasticsearch by generating an enrollment token: ```sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana```. Open Kibana on your browser and paste the token - it will ask for a verification code that you can generate using ```sudo /usr/share/kibana/bin/kibana-verification-code```. 