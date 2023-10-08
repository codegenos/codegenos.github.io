---
title: "Build Multi Platform Docker Images with Buildx"
date: 2023-08-27T13:41:03+03:00
description: "How to multi-platform Docker images from a single Dockerfile, enabling you to create images that can run on different architectures and operating systems."
tags: ["Docker", "Docker Buildx", "Multi Platform Docker", "Raspberry Pi"]
categories: ["Docker", "Docker Buildx"]
ShowToc: false
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

## Docker Buildx
Building multi-platform Docker images allows you to create container images that can run on different CPU architectures and operating systems. Docker introduced support for multi-platform builds with [BuildKit](https://github.com/moby/buildkit). 

[buildx](https://github.com/docker/buildx) is a Docker CLI plugin for extended build capabilities with [BuildKit](https://github.com/moby/buildkit). Using `buildx` with Docker requires Docker engine 19.03 or newer.

## Fix
If you want to create images for different architectures (like x86, ARM, etc.), you would need to maintain separate Dockerfiles or build scripts for each architecture. Or you can use `docker buildx`.

With `docker buildx` you can build images for multiple platforms in a single build process. It allows you to build `multi-platform Docker images` from a `single Dockerfile`, enabling you to create images that can run on different architectures and operating systems.

I'll show you how to build multi-platform Docker images using `docker buildx` that allows you to create images that can run on different architectures (such as x86, ARM, and others) from a single Dockerfile.

Let's start:

1. **Install Docker Buildx:**
To build multi-platform images, you should use Docker Buildx, which is a CLI plugin for building Docker images. You can install it by running the following command:

```bash
docker buildx install
```

2. **Initialize Buildx:**
After you install buildx plugin, initialize `docker buildx` with the platforms you want to target. `--platform` flag specifies multiple platforms.
Create a builder instance that supports multiple platforms. You can create a builder with a specific name like "mybuilder" using the following command:

```bash
docker buildx create --use --name mybuilder --platform linux/amd64,linux/arm64,linux/arm/v7
docker buildx inspect mybuilder --bootstrap
```

The "bootstrap" process takes care of configuring the builder with the necessary settings to enable multi-platform image builds based on the specified platforms. Once the builder is created and bootstrapped, you can use it to build Docker images for those platforms.

You can see the new builder is registered using `docker buildx ls` command:

```bash
docker buildx ls
NAME/NODE       DRIVER/ENDPOINT  STATUS  BUILDKIT             PLATFORMS
mybuilder *     docker-container                              
  mybuilder0    desktop-linux    running v0.12.1              linux/amd64*, linux/arm64*, linux/amd64/v2, linux/amd64/v3, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
```

3. **Build the Multi-Platform Image and Push:**
Now you can build and push the image to the container registry. The `--push` flag is used to push the built image to a container registry. Once your image is built, you can push it to a container registry. Make sure you're authenticated with the registry:

```bash
docker buildx build -t your-image-name:tag --push . -f Dockerfile
```

Or you can the specify the platforms with `--platform` flag during image build:

```bash
docker buildx build -t your-image-name:tag --platform linux/amd64,linux/arm64,linux/arm/v7 --push . -f Dockerfile
```

Replace your-image-name, and tag with the appropriate values for your registry.

You can inspect the image you pushed using `docker buildx imagetools` command. The docker buildx imagetools inspect command is used to inspect and gather information about a Docker image (like a manifest list) that can contain multiple platform-specific images.

You can see multiple platform-specific images in the Manifests list:

```bash
docker buildx imagetools inspect your-image-name:tag

Name:      docker.io/your-image-name:tag
MediaType: application/vnd.oci.image.index.v1+json
Digest:    sha256:911d76b434df65b70eab98ee2e733c0ed861d885a19bf7b49be71ae058595703
           
Manifests: 
  Name:        docker.io/your-image-name:tag@sha256:7ddd1c363bc3f241ac00fc011a13c8a061a4deafc82487c6f3c64571d8708936
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/amd64
               
  Name:        docker.io/your-image-name:tag@sha256:58d47b0e985f31f873ec4b1f0d8c81223af1cb559ba3c1d2fd96966c6ef21d0f
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/arm64
```

When you pull the docker image from your Raspberry Pi 3 Model B, it pulls the image which is built for `linux/arm64`. When you run the docker container, `exit code 139` error will be gone.

Thanks for reading.

Happy coding!
