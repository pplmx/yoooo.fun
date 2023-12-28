---
categories:
    - docker
date: 2023-03-10T17:05:11Z
description: create a specified network to fix the conflicts
keywords: docker,network,container,compose,docker-compose
lastmod: 2023-04-13T17:45:11Z
tags:
    - docker
    - network
    - container
    - compose
title: Docker network conflicts with local networks
---



# Docker Compose Network Conflicts

## Compose default network conflicts with the local network

> To fix this issue, we should config the specified network by compose file.
>
> Here is a sample.

```yaml
version: Compose specification
services:
# ...
networks:
    net1:
        ipam:
            config:
                # if not set, the default gateway is 172.66.99.1
                -   subnet: 172.66.99.0/24
                # or set a gateway
                # -   subnet: 172.34.0.0/16
                #     gateway: 172.34.0.1

```
