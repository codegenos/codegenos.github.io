---
title: "Build Multi Platform Docker Images with Buildx"
date: 2023-08-27T13:41:03+03:00
description: "This article is about the how to multi-platform Docker images from a single Dockerfile, enabling you to create images that can run on different architectures and operating systems."
tags: ["Docker", "Docker Buildx", "Multi Platform Docker", "Raspberry Pi"]
categories: ["Docker", "Docker Buildx"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

## Intro
I developed a `node.js` application and i wanted to run it on `Raspberry Pi` as `docker container`. 

So I built a `Docker image` using `docker build` on my machine (Lubuntu 23.04 x64) and pushed it to the `Docker registry`. 

Then I logged into Raspberry Pi and pulled the image from docker registry and run the docker container. 

But it failed to run with `exit code 139`.

## Problem
Traditionally, when you build a Docker image using `docker build`, it builds the image for the architecture of the machine you're running the `docker build` command on.

In my case I was building an `amd64` Docker image because my machine's architecture was `amd64`. The docker container didn't run on Raspberry Pi 3 Model B because it's architecture is `arm64`.

## Fix
If you want to create images for different architectures (like x86, ARM, etc.), you would need to maintain separate Dockerfiles or build scripts for each architecture. Or you can use `docker buildx`.

With `docker buildx` you can build images for multiple platforms in a single build process. It allows you to build `multi-platform Docker images` from a `single Dockerfile`, enabling you to create images that can run on different architectures and operating systems.

I'll show you how to build multi-platform Docker images using `docker buildx` that allows you to create images that can run on different architectures (such as x86, ARM, and others) from a single Dockerfile.

1. **Install Docker Buildx:**

```bash
docker buildx install
```

2. **Initialize Buildx:**
Initialize `docker buildx` with the platforms you want to target. `--platform` flag specifies multiple platforms:

```bash
docker buildx create --use --name mybuilder --platform linux/amd64,linux/arm64,linux/arm/v7,linux/arm/v8
docker buildx inspect mybuilder --bootstrap
```

3. **Build the Multi-Platform Image and Push:**

The `--push` flag is used to push the built image to a container registry.

```bash
docker buildx build -t your-image-name:tag --push . -f Dockerfile
```

Or you can the specify the platfoms with `--platform` flag:

```bash
docker buildx build -t your-image-name:tag --platform linux/amd64,linux/arm64,linux/arm/v7,linux/arm/v8 --push . -f Dockerfile
```