---
categories:
    - docker
date: 2023-03-10T17:00:42Z
description: build amd64 and arm64 images for multi-platform support
keywords: linux,amd64,arm64,docker
lastmod: 2023-03-10T17:00:42Z
tags:
    - linux
    - amd64
    - arm64
    - docker
title: Build amd64 and arm64 for docker images
---



# docker buildx for multiarch support

## create a builder

because the default builder doesn't support multi-arch building,
create a new builder with docker-container driver and use it

```shell
docker buildx create  --use --name multiarch --driver docker-container
```

if your self-build registry is not https, please touch a `buildkitd.toml`

```toml
[registry."your-registry-domain or ip"]
http = true
insecure = true

```

and then use this config to create a builder

```shell
# if your builder has already existed, delete it and create
docker buildx rm multiarch
docker buildx create  --use --name multiarch --driver docker-container --config ./buildkitd.toml
```

## build the multiarch image and push it to your registry

`docker buildx inspect --bootstrap` to get the supported platform information

```shell
# don't ignore the trailing dot, it's to search the current directory's Dockerfile file
docker buildx build --push --platform linux/amd64,linux/arm64 -t pplmx/demo:v2.0.0 .

# if you wanna to specify a Dockerfile
docker buildx build --push --platform linux/amd64,linux/arm64 -t pplmx/demo:v2.0.0 -f ./Dockerfile
```
