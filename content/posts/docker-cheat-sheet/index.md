---
sys:
  pageId: "2cb1f336-f350-800d-b5ff-fdae7758a78d"
  createdTime: "2025-12-16T07:36:00.000Z"
  lastEditedTime: "2025-12-22T05:41:00.000Z"
  propFilepath: "posts/docker-cheat-sheet/index.md"
title: "Docker Cheat Sheet"
date: "2025-12-22T00:00:00Z"
description: "Commonly used commands in Docker."
tags:
  - "🏷️Docker"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "posts/docker-cheat-sheet"
section: "posts"
---

# Image


## Build Commands


```docker
# Build an image.
docker build -t <image-name>

# Build an Image without using cache.
docker build -t <image-name> . –no-cache

# Create a docker image from a running container.
docker commit <container-name-or-id> <image-name>
```


## Cleanup Commands


```docker
# List images.
docker image ls

# Delete the image.
docker rmi <image-name>

# Remove dangling images.
docker image prune
# Remove dangling & used images.
docker image prune -a

# Remove dangling images. (legacy)
docker images -qf dangling=true | xargs -r docker rmi
```


## Docker Registry


```docker
# Log in to Docker Registry
docker login --username <user>

# Pull a version of an image from a registry
docker pull <image>:<tag>

# Tag an image to a repo
docker tag <image> <user>/<repo>

# Search for an image with docker search "{image-search-text}
"docker search "py"

# Push docker image to docker registry.
# Image name has to be in the form of <docker-registry-address>/<docker-repo>[:<docker-tag>].
# Examples are:
# * docker-repo.local/my-image
# * docker-repo.local/my-image:debug
docker push <image-name>
```


# Container


## Interaction Commands


```docker
# Run (create & start) a named container from the image.
docker run --name <container-name> <image-name>
# Run a container in the background.
docker run -d <image-name>

# Run a container with a port mapping.
docker run -p <host-port>:<container-port> <image-name>

# Run a container in interactive mode.
docker run -it <image-name>
# Run a new command inside an already running container.
docker exec [OPTION] <container-name> <command> [ARGs]

# Stop a container.
docker stop <container-id-or-name>
# Gracefully stop all currently running containers.
# -q is a flag that only list out container IDs.
docker stop $(docker ps -q)
```


### **Run Command Arguments**

- `-detach` , `-d`	在背景中執行並運轉容器。
- `-i` 讓容器開啟 `STDIN` 管道，在虛擬終端機中輸入指令時，可以從 Host 傳入到 Container 之中，並有即時且持續的互動。
- `-t` 會建立一個虛擬終端機(`pseudo-TTY`)讓你可以模擬在容器中使用終端機的環境。
- `--publish` , `-p`  開啟容器的 Ports 映射到 Host。
- `--env` , `-e` 設定環境變數。
- `--name` 為容器命名。
- `--network` 將容器加入特定 Docker 網路群組。
- `--rm` 容器將在指令執行結束後自動停止並刪除。

{{<bookmark
name="深入剖析 docker run 與 docker exec 的 -i 與 -t 技術細節"
link="https://blog.miniasp.com/post/2022/11/22/docker-run-docker-exec-docker-attach-tty-and-stdin"
description="深入了解執行 Docker 容器時，使用的 `-it` 參數以及使用情境">}}


### File Copy


```docker
# Copy files from host to container.
docker cp <host-filepath> <container-id-or-name>:<container-filepath>

# Copy files from container to host.
docker cp <container-id-or-name>:<container-filepath> <host-filepath>

# Export all files in container to host.
docker export <container-id-or-name> -o <filename>
```


## **Inspection Commands**


```docker
# List running containers.
docker ps

# List all containers.
docker ps -a

# List out container applied filter '{key}={value}'
docker ps -f 'name=<some-key-words>'

# Follow the logs of a container.
docker logs -f <container-id-or-name>

# Inspect a running container.
docker inspect <container-id-or-name>
```


## **Cleanup commands**


```docker
# Remove a stopped container.
docker rm <container-id-or-name>
# Remove a running container (kill & rm).
docker rm -f <container-id-or-name>

# Delete all non-running containers
docker container prune
# Or clean up all the unused images, containers, networks
docker system prune

# Kill a container.
docker kill <container-id-or-name>
```


# Referrence


[Software Engineer World-Docker Cheat Sheet](https://sweworld.net/cheatsheets/docker/)


[YIDAS Code Docker 指令集 | 指令速查表](https://code.yidas.com/docker-commands-cli-cheatsheet/)


[Docker CLI | Docker Docs](https://docs.docker.com/reference/cli/docker/)
