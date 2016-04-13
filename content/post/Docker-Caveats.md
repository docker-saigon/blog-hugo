+++
categories = ["Production"]
date = "2016-04-11T20:17:12+07:00"
description = "What you should know about running Docker in Production"
draft = false
image = "/img/fine-print.png"
tags = ["Docker", "Production"]
title = "Docker Caveats"

+++

We can't deny Linux Containers are a very powerful concept combining clever Linux kernel features and Docker's open source tools make containers easily accessible to developers of any background.

At container summit 2016, [Bryan Cantrill](https://twitter.com/bcantrill) eloquently compared the industry disruption this causes, and the issues mass industry adoption entails, to the issues which may show up after [you've taught peasants to read](http://containersummit.io/events/nyc-2016/videos/the-evolving-container-ecosystem) *(about 28 minutes into the linked video of the panel discussion).*

Issues such as: improper usage of the technology and unpleasant surprises due to a poor understanding of the underlying features enabling the technology.

Yesterday, a brilliant Downfall parody made by [Avishai Ish-Shalom](https://twitter.com/nukemberg) highlights some of the surprises & frustrations which may cause shock to those that are unprepared:

{{< tweet 719146833558183936 >}}

In this blog post, we'd like to take a look at each of these statements and deconstruct them for a better understanding of what makes this short so clever, while at the same time - it serves as a great caveat for anyone hoping to get the best out of running Docker in production.

## Isolation

The video starts with what looks like a very popular CI/CD setup using Docker's public image registry, the Docker Hub and its multi-container management tool, Docker-Compose. Although it should be noted that Docker-Compose is still primarily aimed at Development and Testing environments and is probably not suited for larger production deployments, as clearly outlined in the [Docker docs](https://docs.docker.com/compose/production/) at the time of this blog post.

{{< figure src="/img/caveat/caveat01.png" alt="Untrusted Images">}}

{{< figure src="/img/caveat/caveat02.png" alt="Kernel Panic in a Shared Kernel" >}}

**This highlights a first issue of sharing the kernel: reduced reliability and redundancy.**

{{< figure src="/img/caveat/caveat03.png" alt="Isolation my ass!" >}}

We believe, the take-away here should be:

> Containers should not be used without ensuring that reliability and redundancy of every resource is incorporated into the overall design of your infrastructure. 

You may gain back reliability by using shared storage, service orchestration, monitoring and a framework with built-in self-healing features such as [the container rescheduling on node failure"  features added to Swarm](http://container-solutions.com/rescheduling-containers-on-node-failures-with-docker-swarm-1-1/); Or the ingrained concept of "The Reconciliation Loop" in [Kubernetes ReplicaSets](http://kubernetes.io/docs/user-guide/replicasets/).

Although a snarky comment on the above is also included in the video:

{{< figure src="/img/caveat/caveat09.png" alt="Keep it Simple, Stupid" attrlink="https://twitter.com/hashtag/GIFEE" attr="#GIFEE">}}

Later on, another concern related to the implementation of isolation provided by container runtimes is also highlighted: 

{{< figure src="/img/caveat/caveat08.png" alt="Resource Isolation in a Shared Kernel" >}}

Disillusionment comes from treating Docker like magic.

Covered in detail by [Jérôme Petazzoni in his DockerCon EU 2015 presentation](https://www.youtube.com/watch?v=sK5i-N34im8), control groups are integral to what makes up a linux container and fundamental to the resource usage control per process group. A fix for the above complaint is added to Docker 1.11, details can be found in the below Twitter conversation between the video creator and Docker maintainer [@jfrazelle](https://github.com/jfrazelle):

{{< tweet 719326696084606978 >}}

Care is also required surrounding entropy depletion in cloud environments, which is certainly very relevant in shared-kernel scenarios and we may refer to [HAVEGED](https://github.com/gesellix/haveged) as a work-around for this.

## Image Security

The 2nd issue highlighted above was the mis-placed trust in container images pulled from public registries.

{{< figure src="/img/caveat/caveat04.png" alt="Untrusted Images">}}

The very apt "[Sandwich Analogy](http://sobersecurity.blogspot.com.ee/2016/03/containers-are-like-sandwiches.html)" does a great job explaining why using non-official public images from the Docker Hub should be a concern.

> Let's think about Containers in the context of Sandwiches. You can pick up a sandwich. You can look at it, you can tell basically what's going on inside. Are there tomatoes? Lettuce? Ham? Turkey? It's not that hard. There can be things hiding, but for the most part you can get the big details. This is just like a container. Fedora? Red Hat? Ubuntu? It has httpd, great. What about a shell? systemd? Cool. There can be scary bits hidden in there too. Someone decided to replace /bin/sh with a python script? That's just like hiding the olives under the lettuce. What sort of monster would do such a thing!

The security of image contents was big in the news all of 2014 & 2015. Docker has been working diligently to add the required building blocks to fill the gaps. [Content Addressable image layers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#content-addressable-storage) to verify image content against signed manifests, Registry repositories with proper pull validation no longer requiring Image IDs to be treated as secrets, [Nautilus deep inspections](https://blog.docker.com/2015/11/dockercon-eu-2015-docker-universal-control-plane/) on the hub ensuring exposed vulnerabilities are patched in the **official** public Images, [User Namespaces, Seccomp and AppArmor profiles](https://blog.docker.com/2016/02/docker-engine-1-10-security/) as well as [other Security additions](https://blog.docker.com/2015/12/docker-webinar-qa-intro-to-docker-security/) to the Docker Engine, .... 

Refer also to the [Docker docs](https://docs.docker.com/engine/security/) and [the Docker Security Portal](https://www.docker.com/docker-security).

## Docker Defaults

As highlighted in our [Docker Internals](/post/Docker-Internals/) blog post, if your Linux Kernel > 2.6.x - you need to disable the `userland-proxy` on the Docker daemon in favor of Hairpin NAT!

{{< figure src="/img/caveat/caveat13.png" alt="docker-proxy">}}

In general, careful study of the docker defaults is required to ensure the optimal configuration for your environment and use-case. Things such as selecting the appropriate Copy-on-Write Filesystem are all covered in the Docker docs.

## Containers vs VMs

Containers provide significant advantages over Virtual Machines for the use of "Application Packaging" due to the fact that they take a short time to build, are moved around easily and can start and stop very quickly compared to VMs. 

{{< figure src="/img/caveat/caveat05.png" alt="Inception">}}

Unfortunately, in Windows & OSX - virtualisation is required to run the Linux kernel and work with Linux containers. If this is not fully understood, this may cause frustration.

Docker is also improving this with the newest Docker client (which is in private beta at the time of writing). The approach used by the newer Docker clients integrates more deeply with the host operating system which greatly streamlines the developer experience on non-Linux operating systems.

## Distribution & Deployment

The Docker tools not only popularized Container technology, they also included critical shipping functionality making Containers an increasingly popular way to package and deploy code. Container images solve many real-world problems with existing packaging and deployment tools.

{{< figure src="/img/caveat/caveat06.png" alt="Bloated Images">}}

{{< figure src="/img/caveat/caveat07.png" alt="Use Package Managers">}}

However, as containers were being adopted by the masses without differentiating them from the way Virtual Machines tend to be used, images were often shipped with full Linux distributions and countless unnecessary binaries packaged within. This does not only bloat the images, causing slow deployment times, but also increases the attack surface for the application running in production. 

Luckily the community has been adopting slim `Application containers`, using minimal Linux distributions such as Alpine - which is now being [used for all the Official docker images](https://news.ycombinator.com/item?id=11000378) and statically compiled binaries that only rely on the kernel they are built for.

{{< figure src="/img/caveat/caveat14.png" alt="Scalable Apps">}}

Scalability in your App is still up to you and will require you to explore the scenarios enabled by Containers.

The concept of Container [Pods](http://kubernetes.io/docs/user-guide/pods/) encourage the decomposition of applications into even smaller modular, focused, cooperating containers. The isolation provided by containers are sufficient to allow the design of reusable components which lead to more reliable, more scalable and faster to build services than applications built from monolithic containers. We believe these concepts require a change in mindset of what it means to build applications for the cloud. Read more about: [Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html).

But even for existing legacy applications, which may be less "CloudNative", containers enable powerfull deployment patterns such as [The autopilot pattern](https://www.joyent.com/blog/dbaas-simplicity-no-lock-in) pioneered by Joyent.

## Microsoft

{{< figure src="/img/caveat/caveat10.png" alt="Microsoft Containers">}}

Microsoft committed early on to supporting the Docker API for Windows containers with Windows Server 2016. After contributing to ensure the Docker client tools worked well on Windows, implementing Filesystem and Container fundamentals in the Windows Kernel and even open sourcing the Dot Net Core CLR.

{{< figure src="/img/caveat/caveat11.png" alt="Microsoft Containers">}}

Microsoft seems to have gone full-out by extending Project Astoria (an Android emulator) into an impressive Windows SubSystem for Linux (WSL) announced just last week and surprising everyone.

{{< figure src="/img/caveat/caveat12.png" alt="Bash on Windows">}}

The current Linux Kernel API features integrated with Windows however, are targeted at the most common Linux system calls and just enough to [make Windows a more attractive platform for software development](http://arstechnica.com/information-technology/2016/04/why-microsoft-needed-to-make-windows-run-linux-software/) (at the moment).

Integrating these recent events into a wonderfully joyful way, our hats are off to the creator of this video!

[Discuss on Hacker-News](https://news.ycombinator.com/item?id=11477020)

Reference: [Ignore the Hype](http://blog.takipi.com/ignore-the-hype-5-docker-misconceptions-java-developers-should-consider/)