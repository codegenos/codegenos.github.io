---
author: "CodeGenos"
title: "How to Reduce Node.js Docker Image Size?"
date: 2023-08-30T22:20:21+03:00
description: "This article is about reducing Node.js Docker image size from 1GB to 178MB"
tags: ["Docker", "Node.js"]
categories: ["Docker", "Node.js"]
ShowToc: false
ShowBreadCrumbs: true
draft: false
---

I had a web application that is written in Node.js and I wanted to dockerize it. So went to the Node.js official site and found [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp) article. I created the `Dockerfile` as it says in the article. Then I built the docker image with `docker build`, the created image size was `1.09GB`

## Dockerfile which produce 1.09GB docker image

```dockerfile
FROM node:18

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --omit=dev

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```

Bigger images requires disk space and downloading them takes longer time. 1GB size was too big for me and i thought it could be reduced and investigated. The image size can be reduced by the following steps.

### 1. Choose a Smaller Base Image
`jessie`, `buster`, `stretch` and `alpine` images are the smaller images. I've chosen the `alpine` image.

```dockerfile
# Node.js alpine image as the base image
FROM node:18-alpine
```

### 2. Use Multi-Stage Docker Builds

Multi-stage builds are useful to anyone who has struggled to optimize Dockerfiles and then copy only the needed files to run the application into the final image.

```dockerfile
# Node.js alpine image as the base image. This is the build image stage
FROM node:18-alpine as builder

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install app dependencies
RUN npm install

# Bundle app source
COPY . .

# The runtime image stage
FROM node:18-alpine as final

# Copy runtime files from builder image stage
COPY --from=builder /app/ /app/

# Set the working directory in the container
WORKDIR /app

# Expose the port your app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["node", "server.js"]
```

### 3. Copy only the required files

Reduce the size of the image by making sure to only `COPY` the required files.

```dockerfile
# Node.js alpine image as the base image. This is the build image stage
FROM node:18-alpine as builder

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install app dependencies
RUN npm install

# Bundle app source
COPY . .

# The runtime image stage
FROM node:18-alpine as final

# Copy only the required files from builder image stage
COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/server.js /app/server.js

# Set the working directory in the container
WORKDIR /app

# Expose the port your app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["node", "server.js"]
```

### 4. Install Production Dependencies Only or Omit devDependencies

`ENV NODE_ENV=production` causes to install only production on npm install.

`RUN npm ci --omit=dev` omits the dev dependencies.

```dockerfile
# Node.js alpine image as the base image. This is the build image stage
FROM node:18-alpine as builder

# Bundle app source
COPY . /app

# Make node environment production
ENV NODE_ENV=production
WORKDIR /app

# Omit dev dependencies during npm install
RUN npm ci --omit=dev

# The runtime image stage
FROM node:18-alpine as final

# Copy only the required files from builder image stage
COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/server.js /app/server.js

# Set the working directory in the container
WORKDIR /app

# Expose the port your app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["node", "server.js"]
```

### Result
The final image size is reduced to `178MB`.

