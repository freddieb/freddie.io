+++
title = "Auto Deploying Docker Containers with Watchtower"
date = "2019-08-18"
author = ""
cover = ""
tags = ["docker", "docker-compose", "watchtower", "continuous-deployment"]
description = "How to use Watchtower to quickly setup automatic deployment of your Docker images. The setup is super simple, and the servies used are all free."
showFullContent = false
+++

## The Problem
For ages, I would SSH into my server, `git pull` my code and then run `docker build && docker down && docker up -d` every time I made a new change. It was horrendously time consuming! Sometimes I’d be debugging a production server for ages only to realise I hadn’t updated the server. 🤦

![Getting angry with my computer](/img/angry_computer.gif)

This article is a step by step tutorial on how to achieve an automated deployment pipeline for your Docker images. It will drastically improve your deployment time for new changes and ensure your server is always up to date with the latest version of your code.

## Tools
- Docker & Docker Compose — the bread and butter! 🍞
- Watchtower - the magic  🔮
- Github/Gitlab/BitBucket repository 🤖
- Docker Registry (DockerHub)* 🐳

\* If you are using GitLab, you can use their own docker registry that comes with your account. 

## Assumptions
This tutorial assumes you have a working understanding of Docker and at least one Docker container ready to deploy.

## Code
All of the code referenced (and supporting files) can be viewed in this repo:  [https://github.com/freddieb/docker-watchtower-tutorial](https://github.com/freddieb/docker-watchtower-tutorial) . If you don’t currently have a Docker image, the repo contains a simple api server that can be used along side the material in this tutorial.

## The Steps!
### Add your image to DockerHub
First, make sure the project is available via GitHub, including the `Dockerfile`.

Then, add the Docker image to your Docker registry by creating a Docker repository and connecting it to your GitHub account. Replace the values shown below with your own, but remember to ensure the build rules are set so that the tag is always `latest`. This ensures the image on the server can always be updated.  Then click ‘Create & Build’!

![](/img/DockerHub_CreateRepo_Screenshot.png)


### Create a docker compose file
If you do not already have one, you will need to create a `docker-compose.yaml` config file. Docker Compose allows multiple Docker containers to be managed at once, including building, running and stopping the containers. The `docker-compose.yaml` file is the configuration that explains how the containers should be setup and communicate (or not) with each other.

The example `docker-compose.yaml` below orchestrates two containers. One container is a basic web server, and the second is the Watchtower container that will monitor the web server and update it when required.

```
version: '3'

services:
  api:
    image: index.docker.io/freddieb/example-api:latest
    restart: unless-stopped
    ports:
    - "80:3000"

  # Watches the docker registry for updates on the containers
  # Must run docker login initially on host machine
  watchtower:
    image: containrrr/watchtower
    restart: unless-stopped
    volumes:
      # Allows container to interact with Docker API
      - /var/run/docker.sock:/var/run/docker.sock
    # Check for new images every 90 seconds
    command: --interval 90 
```

If you already have a `docker-compose.yaml` file, the watchtower service can be appended to your existing setup.

IMPORTANT: If your DockerHub registry will be private, you need to add the following volume (in the `volumes` array) to give Watchtower access to your login credentials.
```
# Docker (registry) login credentials
- $HOME/.docker/config.json:/config.json
```

### Server setup
SSH into your server and check that you have Docker Compose installed by checking the current version.
```
$ docker-compose --version
```

If your Docker repository was set to private, you will need to login by running:
```
$ docker login
```

Then, copy your `docker-compose.yml` file onto the server.
```
$ scp docker-compose.yaml <user>@<server_address>:
```

Finally, run and view the compose logs. It should be running as you’d expect! If its not working, double check your firewall is configured to allow traffic on the port your container exposes to the host.
```
$ docker-compose up -d
$ docker-compose logs -f
```

Once it’s up and running, push a change to the GitHub repository and watch it take affect on your server! You can monitor the process of the update by seeing the build progress in DockerHub.

- - - -

I hope this was useful, if you have any questions let me know and I’ll do my best to assist!

