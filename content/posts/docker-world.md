---
title: Let's deeply understand how to run a container
date: 2021-03-31T14:48:04+08:00
slug: 8b6e7fcca8d05957a2776ab6e4b26c4c
draft: false
lastmod: 2021-03-31T15:31:10+08:00
categories: [docker]
tags: [docker, container]
keywords: container, docker, dockerd, containerd, containerd-shim, runc
description: Docker Architecture
---
# Docker

[Referenced from here.](https://www.docker.com/blog/introducing-docker-engine-18-09/)

![Introducing Docker Engine 18.09 - Docker Blog](assets/DockerEngineDiagram-1.png)

[Referenced from here.](https://mkdev.me/en/posts/the-tool-that-really-runs-your-containers-deep-dive-into-runc-and-oci-specifications)

![Differ Container](assets/differ-container.png)

[Referenced from here.](https://medium.com/@avijitsarkar123/docker-and-oci-runtimes-a9c23a5646d6)

![Docker and OCI Runtimes.](assets/docker_oci.png)

## Components

- dockerd
- containerd
- containerd-shim
- runc

### runc

`CLI tool` for spawning and running containers according to the `OCI specification`.

### containerd-shim

The shim allows for `daemonless` containers. It basically sits as the parent of the container's process to facilitate a few things.

- First it allows the runtimes(i.e. `runc`) to exit after it starts the container. This way we don't have to have the long running runtime processes for containers. When you start `mysql` you should only see the `mysql` process and the shim.
- Second it keeps the STDIO and other fds open for the container incase `containerd` and/or docker both die. If the shim was not running then the parent side of the pipes or the TTY master would be closed and the container would exit.
- Finally it allows the container's exit status to be reported back to a higher level tool like docker without having the be the actual parent of the container's process and do a `wait4`.

### containerd

**containerd** was introduced in Docker 1.11 and since then took main responsibility of managing containers life-cycle. `containerd` is the *executor for containers*, but has a wider scope than *just executing* containers. So it also take care of:

- Image push and pull
- Managing of storage
- Of course executing of Containers by calling **runc** with the right parameters to run containers...
- Managing of network primitives for interfaces
- Management of network namespaces containers to join existing namespaces

### dockerd

The Docker daemon - **dockerd** listens for Docker API requests and manages host's Container life-cycles by utilizing **containerd**.

`dockerd` can listen for Docker Engine API requests via three different types of Socket: `unix`, `tcp`, and `fd`.

By default, a `unix` domain socket is created at `/var/run/docker.sock`, requiring either root permission, or docker group membership.

On [Systemd](http://alexander.holbreich.org/tag/systemd/) based systems, you can communicate with the daemon via `Systemd` socket activation, use `dockerd -H fd://`.

## Workflow among the above components

```bash
❯ docker --version
Docker version 20.10.5, build 55c4c88

❯ sudo docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS       PORTS     NAMES
05ebb62bc655   nginx     "/docker-entrypoint.…"   13 days ago   Up 13 days   80/tcp    nginx2
7f3fa77ddad8   nginx     "/docker-entrypoint.…"   13 days ago   Up 13 days   80/tcp    nginx1

❯ ps -ef --forest | grep -v " --color=auto" | grep -A3 -E "dockerd|containerd"
root         714       1  0 Mar17 ?        00:17:12 /usr/bin/containerd
root        1931       1  0 Mar17 ?        00:02:53 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root        2139       1  0 Mar17 ?        00:01:04 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 7f3fa77ddad85e82619b21d6fd9bde8c6fa7cce9e1c063b4f18f258c1206b1e4 -address /run/containerd/containerd.sock
root        2163    2139  0 Mar17 ?        00:00:00  \_ nginx: master process nginx -g daemon off;
systemd+    2217    2163  0 Mar17 ?        00:00:00      \_ nginx: worker process
root        2240       1  0 Mar17 ?        00:01:05 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 05ebb62bc6557c76f9d4494bbd2262e9fa7f91c3a0bad14677455a158e949f75 -address /run/containerd/containerd.sock
root        2261    2240  0 Mar17 ?        00:00:00  \_ nginx: master process nginx -g daemon off;
systemd+    2319    2261  0 Mar17 ?        00:00:00      \_ nginx: worker process

=================================

❯ docker --version
Docker version 19.03.15, build 99e3ed8919

❯ docker container ls
CONTAINER ID    IMAGE    COMMAND                  CREATED           STATUS           PORTS     NAMES
cbb233ea0045    nginx    "/docker-entrypoint.…"   11 minutes ago    Up 11 minutes    80/tcp    nginx2
fa3468d6e89a    nginx    "/docker-entrypoint.…"   11 minutes ago    Up 11 minutes    80/tcp    nginx1

❯ ps -ef --forest | less
root      184283       1  0 14:10 ?        00:00:00 /usr/bin/containerd
root      184493  184283  0 14:11 ?        00:00:00  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/fa3468d6e89a7ddcbd67a7049b2fd1771555c445ba6e8795a4634cb4795ecdd6 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root      184509  184493  0 14:11 ?        00:00:00  |   \_ nginx: master process nginx -g daemon off;
101       184564  184509  0 14:11 ?        00:00:00  |       \_ nginx: worker process
root      184595  184283  0 14:11 ?        00:00:00  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/cbb233ea004589877970ee3b4bcd08672370c159720617c29c31b943e4a5be3c -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root      184611  184595  0 14:11 ?        00:00:00      \_ nginx: master process nginx -g daemon off;
101       184663  184611  0 14:11 ?        00:00:00          \_ nginx: worker process
root      184291       1  0 14:10 ?        00:00:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

=================================

❯ podman version
Version:      2.2.1
API Version:  2
Go Version:   go1.14.12
Built:        Mon Feb 22 12:51:35 2021
OS/Arch:      linux/amd64

❯ podman container ls
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS            PORTS   NAMES
2fed78dd707e  docker.io/library/nginx:latest  nginx -g daemon o...  2 minutes ago  Up 2 minutes ago          nginx2
75103237f3d5  docker.io/library/nginx:latest  nginx -g daemon o...  2 minutes ago  Up 2 minutes ago          nginx1

❯ runc list
ID                                                                 PID         STATUS      BUNDLE                                                                                                                     CREATED                          OWNER
2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9   188922      running     /var/lib/containers/storage/overlay-containers/2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9/userdata   2021-03-31T06:36:31.164181537Z   root
75103237f3d5f8d78f1d34cd32747c083f8d59eb5df4d09e3e68ab8279fcf832   188806      running     /var/lib/containers/storage/overlay-containers/75103237f3d5f8d78f1d34cd32747c083f8d59eb5df4d09e3e68ab8279fcf832/userdata   2021-03-31T06:36:25.812499602Z   root

❯ runc state 2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9
{
  "ociVersion": "1.0.2-dev",
  "id": "2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9",
  "pid": 188922,
  "status": "running",
  "bundle": "/var/lib/containers/storage/overlay-containers/2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9/userdata",
  "rootfs": "/var/lib/containers/storage/overlay/9fde6f2ab9dee9701adce3862803afe893c009269e74c0c77e18c4454c9184d1/merged",
  "created": "2021-03-31T06:36:31.164181537Z",
  "annotations": {
    "io.container.manager": "libpod",
    "io.kubernetes.cri-o.Created": "2021-03-31T14:36:30.807867268+08:00",
    "io.kubernetes.cri-o.TTY": "false",
    "io.podman.annotations.autoremove": "FALSE",
    "io.podman.annotations.init": "FALSE",
    "io.podman.annotations.privileged": "FALSE",
    "io.podman.annotations.publish-all": "FALSE",
    "org.opencontainers.image.stopSignal": "3"
  },
  "owner": ""
}# 

❯ ps -ef --forest
root      188797       1  0 14:36 ?        00:00:00 /usr/bin/conmon --api-version 1 -c 75103237f3d5f8d78f1d34cd32747c083f8d59eb5df4d09e3e68ab8279fcf832 -u 75103237f3d5f8d78f1d34cd32747c083f8d59eb5df4d09e3e68ab8279fcf832 -r /usr/bin/runc -b /var/lib/containers/storage/overlay-containers/75103237f3d5f8d78f1d34cd32747c08/run/containers/storage/overlay-containers/75103237f3d5f8d78f1d34cd32747c083f8d59eb5df4d09e3e68ab8279fcf832/userdata/conmon.pid --exit-command /usr/bin/podman --exit-command-arg --root --exit-command-arg /var/lib/containers/storage --exit-command-arg --runroot --exit-command-arg /var/run/containers/storage --exit-comm
root      188806  188797  0 14:36 ?        00:00:00  \_ nginx: master process nginx -g daemon off;
101       188842  188806  0 14:36 ?        00:00:00      \_ nginx: worker process
root      188913       1  0 14:36 ?        00:00:00 /usr/bin/conmon --api-version 1 -c 2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9 -u 2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9 -r /usr/bin/runc -b /var/lib/containers/storage/overlay-containers/2fed78dd707e865d4995f2d80dd9ee78/run/containers/storage/overlay-containers/2fed78dd707e865d4995f2d80dd9ee7830776e8adfe62f2b5b2754fa8b950be9/userdata/conmon.pid --exit-command /usr/bin/podman --exit-command-arg --root --exit-command-arg /var/lib/containers/storage --exit-command-arg --runroot --exit-command-arg /var/run/containers/storage --exit-comm
root      188922  188913  0 14:36 ?        00:00:00  \_ nginx: master process nginx -g daemon off;
101       188955  188922  0 14:36 ?        00:00:00      \_ nginx: worker process


```