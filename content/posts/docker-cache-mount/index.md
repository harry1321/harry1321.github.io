---
sys:
  pageId: "2c61f336-f350-80e4-b52c-cdc6107b8e89"
  createdTime: "2025-12-11T10:13:00.000Z"
  lastEditedTime: "2025-12-11T10:22:00.000Z"
  propFilepath: "posts/docker-cache-mount/index.md"
title: "使用 Cache Mount 加速 Docker Build"
date: "2025-12-11T00:00:00Z"
description: "安裝應用套件時使用 cache mount，幫助 Docker 重複使用快取的檔案加快建置過程。"
tags:
  - "🏷️Docker"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "posts/docker-cache-mount"
section: "posts"
---

# Cache Mount


在 Docker 映像檔的建置過程中，有許多因素會影響到建置的速度和最終的容量。例如，Dockerfile 中指令的多寡和排列順序，會直接影響所生成的映像檔層數。因此，減少和整合 Dockerfile 的每一行指令，是優化映像檔的基本技巧之一。


此外，Docker 在建置時，會檢查 Dockerfile 中的指令和所依賴的檔案，是否自上次建置以來有所更改，藉此決定是否需要重新建置該層。因此，如何善用快取機制，避免重複建置未變動的層或檔案，進而有效縮短建置時間，是改善建置流程的一個重要技術。


本篇文章要介紹的是，屬於 Buildx 的快取機制，用來改善安裝應用程式套件的流程。這個機制稱為 cache mount (快取掛載)，是一個持久 (persistent) 的快取，可以在建置過程被多次重複使用。即使 Docker 偵測到變動，要重新建置層，也只會下載新增或變更的套件。


Cache mounts 的使用方式很簡單，就是在 Dockerfile 安裝套件的 `RUN` 指令加上 `--mount=type=cache,target=<path>`，以 python pip 安裝為示範。

其中 `--no-cache-dir` 告訴 pip 不要將它的快取寫入到最終的 Docker 映像層中，確保安裝過程中間產生的快取文件不會被提交到最終的映像，可以減少映像的體積。

```docker
# Use a cache mount for pip
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir -r requirements.txt
```


`target=<path>` 是指向 builder 快取的路徑，這個路徑通常是 packager manager 使用的快取路徑，如此一來才能正確載入快取。如果這個套件快取會同時被多個建置取用，需要設定 `sharing=locked`  參數來避免建置衝突，使用 `private` 可以將不同的建置快取獨立儲存。


# uv 與 Docker 的結合


針對使用 uv 來管理 Python 套件，官方也提供明確的指引，包含 [single build](https://github.com/astral-sh/uv-docker-example/blob/main/Dockerfile) 和 [multi-stage build](https://github.com/astral-sh/uv-docker-example/blob/main/multistage.Dockerfile)。這裡擷取部分的 Dockerfile 來介紹。有兩種方式可以在 Docker 中使用 uv，一種是直接拉取已經建置好的 base image，另一種是手動安裝或下載二進位檔案。


```docker
# Use a Python image with uv pre-installed
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim
...
```


```docker
# Copying the binary from the official distroless Docker image
FROM python:3.12-slim-bookworm
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
...
```


官方提供的檔案中，將套件安裝分成兩個獨立的 `RUN` 步驟，目的是為了充分利用 Docker 的層級快取機制。


第一次 `uv sync` 只安裝套件依賴，`--no-install-project`：不安裝當前這個 Python 專案本身，即應用程式程式碼。只要 `uv.lock` 和 `pyproject.toml` 這兩個文件內容沒有改變，Docker 在下次建置時就會重用這個快取層，而不會重新下載和安裝所有套件。


第二次 `uv sync` 才安裝專案，原因在於應用程式的原始碼可能會經常發生變化，如果合併到同一個 步驟中，任何程式碼的變動會導致快取被破壞，並重新安裝所有套件。


```docker
...
# Install the project's dependencies using the lockfile and settings
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project

# Then, add the rest of the project source code and install it
# Installing separately from its dependencies allows optimal layer caching
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked
...
```


{{<bookmark
name="如何在 Docker 中使用 uv"
link="https://docs.astral.sh/uv/guides/integration/docker/"
description="A complete guide to using uv in Docker to manage Python dependencies while optimizing build times and image size via multi-stage builds, intermediate layers, and more">}}


# 參考來源


[Optimize cache usage in builds](https://docs.docker.com/build/cache/optimize/#use-cache-mounts)


[Dockerfile reference RUN --mount=type=cache](https://docs.docker.com/reference/dockerfile/#run---mounttypecache)


[優化加速 docker build 的秘訣 —— 使用 Cache mounts](https://myapollo.com.tw/blog/docker-cache-mounts/)
