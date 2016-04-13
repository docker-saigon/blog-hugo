+++
categories = ["Swarm", "Docker-Machine"]
date = "2016-03-30T04:38:00+07:00"
description = "Swarming the Birthday App"
draft = true
image = "/img/docker-swarm.jpg"
tags = ["swarm", "swarm-week", "#dockerbday", "docker-machine", "Windows", "hyperv", "Powershell"]
title = "Swarm week 2016 - Part 2: Deploying Micro Services"
+++

Docker's 3th birthday has been celebrated all over the world ([including](https://twitter.com/docker_saigon/status/713652799704162304) [Saigon](https://twitter.com/docker_saigon/status/713291864200187905))

In this post we will look at deploying and scaling the [birthday app](https://github.com/docker/docker-birthday-3) on top of the Swarm cluster from our previous post.

As the [new Docker version for Windows](https://beta.docker.com/) is still in closed beta, we will continue to use our custom `Docker-Machine` + `Hyper-V` + `Boot2Docker` setup. We hope to create an updated post as soon as possible.

# Re-configure our Private Registry as a HUB-proxy

## 1. Create the Machine

## 2. Modify the Registry config

Get the default config
```
docker run -it --rm --entrypoint cat registry:2 \
/etc/docker/registry/config.yml > /data/config.yml
```

## 3. Using Interlock with TLS for scaling