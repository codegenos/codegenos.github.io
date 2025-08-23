---
author: "CodeGenos"
title: "Build Multi-Platform Docker Images with Docker Buildx"
date: 2023-08-27T13:41:03+03:00
lastmod: 2025-08-23T00:00:00+03:00
description: "Learn how to build multi-platform (multi-arch) Docker images with Docker Buildx and BuildKit from a single Dockerfile for amd64, arm64, and ARMv7 (Raspberry Pi)."
tags: ["Docker", "Docker Buildx", "Multi Platform Docker", "Raspberry Pi", "Multi-arch", "BuildKit", "QEMU", "ARMv7"]
categories: ["Docker", "Docker Buildx"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

## Intro
I developed a `Node.js` application and wanted to run it on a `Raspberry Pi` in a `Docker` container.

I built a Docker image on my x64 laptop (Ubuntu) and pushed it to Docker Hub. On the Raspberry Pi, I pulled the image and tried to run the container — but it failed with `exit code 139`.

## Problem
By default, `docker build` creates an image for the architecture of the machine you build on. In my case, I produced an `amd64` image on an `amd64` machine. A Raspberry Pi 3 Model B typically runs a 32‑bit OS, so it expects `linux/arm/v7` (armhf). It will only use `linux/arm64` if you run a 64‑bit OS. The architecture mismatch caused the runtime error.

## Docker Buildx in a nutshell
Multi‑platform (multi‑arch) images let you ship one image tag that works on multiple CPU architectures. Docker added this capability with [BuildKit](https://github.com/moby/buildkit), and the `buildx` CLI provides an easy interface for it.

## Prerequisites
- Docker Engine 19.03+ (Buildx is included in modern Docker versions)
- Access to a container registry (e.g., Docker Hub) and permission to push

## TL;DR / Quick start
Quick steps to build and push a multi-arch image from one Dockerfile:

```bash
# Verify Buildx (install if missing)
docker buildx version

# Create and bootstrap a multi-platform builder
docker buildx create --use --name mybuilder --platform linux/amd64,linux/arm64,linux/arm/v7
docker buildx inspect mybuilder --bootstrap

# Login to your registry
docker login

# Build and push a multi-arch image
docker buildx build -t <registry>/<repo>:<tag> \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --push -f Dockerfile .

# Verify manifest
docker buildx imagetools inspect <registry>/<repo>:<tag>
```

Note: Raspberry Pi 3 with a 32-bit OS will pull `linux/arm/v7`; with a 64-bit OS it pulls `linux/arm64` automatically.

## Solution
You can build and push a single tag that contains multiple architectures, from one Dockerfile.

To build multi-platform images, you should use Docker Buildx, which is a CLI plugin for building Docker images.

1) Verify Buildx is available

```bash
docker buildx version
```

If it is not installed can install it by running the following command:

```bash
docker buildx install
```

2) Create and bootstrap a multi‑platform builder

```bash
docker buildx create --use --name mybuilder --platform linux/amd64,linux/arm64,linux/arm/v7
docker buildx inspect mybuilder --bootstrap
```

Notes:
- The `--platform` list declares what the builder supports. You still select target platforms at build time.
- Bootstrap configures the builder and sets up emulation for cross‑building.

You can see the new builder is registered using `docker buildx ls` command:

```bash
docker buildx ls
NAME/NODE       DRIVER/ENDPOINT  STATUS  BUILDKIT             PLATFORMS
mybuilder *     docker-container                              
  mybuilder0    desktop-linux    running v0.12.1              linux/amd64*, linux/arm64*, linux/amd64/v2, linux/amd64/v3, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
```

3) Build and push a multi‑platform image

Make sure you are authenticated first:

```bash
docker login
```

Now build and push with explicit platforms:

```bash
docker buildx build -t <registry>/<repo>:<tag> \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --push -f Dockerfile .
```

## Verify the image manifest
Use `imagetools` to confirm you have per‑platform manifests:

```bash
docker buildx imagetools inspect <registry>/<repo>:<tag>
```

You should see entries such as `linux/amd64`, `linux/arm64`, and `linux/arm/v7`. When you pull this tag on Raspberry Pi:
- On a 32‑bit OS (common on Pi 3), Docker pulls `linux/arm/v7`.
- On a 64‑bit OS, Docker pulls `linux/arm64`.

The multiple platform-specific images in the Manifests list:

```bash
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

The previous `exit code 139` or "exec format error" should be resolved once the correct architecture is used.

## Troubleshooting
- Exit code 139 or "exec format error": Ensure your image includes the correct platform for the device OS (`arm/v7` for 32‑bit, `arm64` for 64‑bit).
- Base image mismatch: Use base images that publish multi‑arch variants (e.g., `node:lts-alpine`), or specify an `--platform` on `FROM` in multi‑stage builds if needed.
- Slow cross‑builds: Emulation can be slower. For speed, consider native builders per architecture or remote builders.
- Buildx not found: Upgrade Docker or install the Buildx plugin; then re‑run the steps above.

## FAQ
- What is Docker Buildx?
  - A Docker CLI plugin that uses BuildKit to build images, including multi‑platform images, efficiently.
- Do I need QEMU for multi‑arch builds?
  - The docker‑container driver configures binfmt/QEMU during `--bootstrap` so you can cross‑build without manual setup.
- Which platform should I target for Raspberry Pi 3 Model B?
  - Usually `linux/arm/v7` (32‑bit OS). Use `linux/arm64` only if you run a 64‑bit OS on the Pi.

## Conclusion
With Docker Buildx you can ship one tag that works across `amd64`, `arm64`, and `arm/v7` from a single Dockerfile.

You may also like: [How to Reduce Node.js Docker Image Size](/posts/how-to-reduce-node-js-docker-image-size/)

Thanks for reading.

Happy coding!
