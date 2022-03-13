---
title: Learning Docker
date: 2020-06-05T16:42:36+08:00
lastmod: 2020-11-23T10:00:59+08:00
slug: 6195694198c849fbb661f4746beb101b
draft: false
categories: [general]
tags: [docker,cli]
keywords: docker cli, common cli
description: Let's start to learn Docker.
---
# Docker General Knowledge

## Common CLi

```bash
# list all images in local
# -a, i.e. --all [Show all images (default hides intermediate images)]
docker image ls -a

# list all images without intermediate images
docker image ls

# list all containers in local(running or stopped)
docker container ls -a

# list all running containers
docker container ls
# Key details, you can refer to the last of this article.
docker container ls --format 'table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}'
docker container ls --format 'table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}'

# pull image from dockerhub(It can be with tag, e.g. latest)
docker image pull centos
docker image pull centos:latest
docker image pull purplemystic/centos

# create a container named my_centos by the latest centos image
# -i, i.e. --interactive
# -t, i.e. --tty
docker container run --privileged -dit --name my_centos centos init

# enter in a docker container
docker container exec -it my_cenos bash

# stop a started container
docker container stop my_centos

# start a stopped container
docker container start my_centos

# kill a container
docker container kill my_centos

# remove all stopped containers
docker container prune

# remove all unused images
docker image prune

# container to host
# copy a test.log in container my_centos /var/tmp to current dir
docker container cp my_centos:/var/tmp/test.log .

# host to container
docker container cp test.log my_centos:/var/tmp

```

## Docker commit

```bash
# save container to a image with your changes
docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

# e.g.
docker container commit my_centos purplemystic/centos
docker container commit my_centos purplemystic/centos:1.2
docker container commit my_centos purplemystic/centos:999
```

`Options`

| Name, shorthand  | Default | Description                                                  |
| ---------------- | ------- | ------------------------------------------------------------ |
| `--author , -a`  |         | Author (e.g., “John Hannibal Smith [hannibal@a-team.com](mailto:hannibal@a-team.com)”) |
| `--change , -c`  |         | Apply Dockerfile instruction to the created image            |
| `--message , -m` |         | Commit message                                               |
| `--pause , -p`   | `true`  | Pause container during commit                                |

## How to SSH among multiple container

```bash
# create three compute node
docker container run -dit --privileged --name compute1 centos init

docker container run -dit --privileged --name compute2 centos init

docker container run -dit --privileged --name compute1 centos init

# login to three nodes separately to run the following command
# step1: to ensure sshd service is active
dnf install -y openssh openssh-server

# step2: to ensure you know user and password(set user root's password to 'root')
echo "root:root" | chpasswd
```

---

Now, you can access one container from another container, like as follows:

```bash
[root@937fa919cd2a /]# ssh root@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ECDSA key fingerprint is SHA256:cBgcL8pqvWncp8Wn0ky7beHWC2ZFNWlR8UXK0roK7Mg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ECDSA) to the list of known hosts.
root@172.17.0.2's password:
[root@0ea4fead9a24 ~]#
```

## Docker network

```bash
# list network
λ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
21a5e8b305c4        bridge              bridge              local
e6aafe917f13        host                host                local
d2de5c3c084c        none                null                local

# check network details
λ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "21a5e8b305c415ab903d86689778d91c7968ab9ab350e0f0885be8a81d0095cc",
        "Created": "2020-06-08T04:21:05.390246004Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0ea4fead9a248c57d0d5053198ea5ba5eedd32f6636dd834d763dddec7bce9fd": {
                "Name": "compute1",
                "EndpointID": "3165851a116e074f7564411a351e74570c164af5610ebf8f45183840eb869199",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "937fa919cd2aacc47034648f739c29fec00a09e14f8363a8a68d5ac3ce786e15": {
                "Name": "compute2",
                "EndpointID": "22e1551440dcdd913dc59612d9757b846a1b7c977f0f836bf5b826ef4da13bab",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

# create a network(default create a network whose network type is bridge)
docker networ create br-int

# create a host network
docker network create --driver host host-net

# --driver
- bridge
- host
- overlay
- macvlan
- none
- Network Plugins
```

### Network drivers

Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality:

- `bridge`: The default network driver. If you don’t specify a driver, this is the type of network you are creating. **Bridge networks are usually used when your applications run in standalone containers that need to communicate.** See [bridge networks](https://docs.docker.com/network/bridge/).
- `host`: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly. `host` is only available for swarm services on Docker 17.06 and higher. See [use the host network](https://docs.docker.com/network/host/).
- `overlay`: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers. See [overlay networks](https://docs.docker.com/network/overlay/).
- `macvlan`: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. Using the `macvlan` driver is sometimes the best choice when dealing with legacy applications that expect to be directly connected to the physical network, rather than routed through the Docker host’s network stack. See [Macvlan networks](https://docs.docker.com/network/macvlan/).
- `none`: For this container, disable all networking. Usually used in conjunction with a custom network driver. `none` is not available for swarm services. See [disable container networking](https://docs.docker.com/network/none/).
- [Network plugins](https://docs.docker.com/engine/extend/plugins_services/): You can install and use third-party network plugins with Docker. These plugins are available from [Docker Hub](https://hub.docker.com/search?category=network&q=&type=plugin) or from third-party vendors. See the vendor’s documentation for installing and using a given network plugin.

## Network driver summary

- **User-defined bridge networks** are best when you need multiple containers to communicate on the same Docker host.
- **Host networks** are best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated.
- **Overlay networks** are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.
- **Macvlan networks** are best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address.
- **Third-party network plugins** allow you to integrate Docker with specialized network stacks.

## Filter and Format

```text
Filters
       Filter output based on these conditions:
          - ancestor=(<image-name>[:tag]|<image-id>| <image@digest>)
            containers created from an image or a descendant.
          - before=(<container-name>|<container-id>)
          - expose=(<port>[/<proto>]|<startport-endport>/[<proto>])
          - exited=<int> an exit code of <int>
          - health=(starting|healthy|unhealthy|none)
          - id=<ID> a container's ID
          - isolation=(default|process|hyperv) (Windows daemon only)
          - is-task=(true|false)
          - label=<key> or label=<key>=<value>
          - name=<string> a container's name
          - network=(<network-id>|<network-name>)
          - publish=(<port>[/<proto>]|<startport-endport>/[<proto>])
          - since=(<container-name>|<container-id>)
          - status=(created|restarting|removing|running|paused|exited)
          - volume=(<volume name>|<mount point destination>)

Format
       The formatting option (--format) pretty-prints container output using a
       Go template.

       Valid placeholders for the Go template are listed below:
          - .ID           - Container ID.
          - .Image        - Image ID.
          - .Command      - Quoted command.
          - .CreatedAt    - Time when the container was created.
          - .RunningFor   - Elapsed time since the container was started.
          - .Ports        - Exposed ports.
          - .Status       - Container status.
          - .Size         - Container disk size.
          - .Names        - Container names.
          - .Labels       - All labels assigned to the container.
          - .Label        - Value of a specific label for this container.
                            For example '{{.Label "com.docker.swarm.cpu"}}'.
          - .Mounts       - Names of the volumes mounted in this container.
          - .Networks     - Names of the networks attached to this container.
```

## FYI

### Container 

> docker container my_command

- `create `
    - Create a container from an image.
- `start `
    - Start an existing container.
- `run `
    - Create a new container and start it.
- `ls `
    - List running containers.
- `inspect `
    - See lots of info about a container.
- `logs `
    - Print logs.
- `stop `
    - Gracefully stop running container.
- `kill `
    - Stop main process in container abruptly.
- `rm`
    - Delete a stopped container.

### Image

> docker image my_command

- `build `
    - Build an image.
- `push` 
    - Push an image to a remote registry.
- `ls` 
    - List images.
- `history` 
    - See intermediate image info.
- `inspect` 
    - See lots of info about an image, including the layers.
- `rm` 
    - Delete an image.