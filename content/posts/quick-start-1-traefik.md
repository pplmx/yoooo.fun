---
categories:
    - docker
date: 2023-04-13T17:25:36Z
description: how to configure custom domain for traefik dashboard
keywords: docker,traefik,proxy,custom domain
lastmod: 2023-11-24T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
title: 'Quick Start: Traefik Dashboard with Custom Domain'
---



# Quick Start: Traefik

## Preparation

```shell
# create dirs and empty files
mkdir -p traefik/dynamic-conf && cd traefik && touch compose.yml traefik.yml dynamic-conf/self.yml
```

### compose.yml

```yaml
version: Compose specification

services:
    traefik:
        image: traefik:3.0
        ports:
            - "80:80"
        environment:
            - TZ=Asia/Shanghai
        volumes:
            # /traefik.yml and /etc/traefik/traefik.yml are both available.
            - "./traefik.yml:/etc/traefik/traefik.yml"
            # dynamic-conf dir is self-defined
            - "./dynamic-conf:/etc/traefik/dynamic-conf"
            - "/var/run/docker.sock:/var/run/docker.sock:ro"
        networks:
            - traefik-net

networks:
    traefik-net:
        name: traefik-net
        ipam:
            config:
                -   subnet: 172.16.238.0/24

```

### traefik.yml

```yaml
### Static Configuration
log:
    level: INFO
api:
    insecure: true
    dashboard: true
entryPoints:
    web:
        address: :80
providers:
    file:
        directory: /etc/traefik/dynamic-conf
        watch: true

```

### self.yml in dir dynamic-conf

```yaml
### Dynamic Configuration
http:
    routers:
        dashboard:
            rule: Host(`traefik.x.internal`)
            service: api@internal

```

## Config DNS domain parse

If you have DNS server, please reference the DNS server guide to config it

If not
    - using the unix-like system: edit the `/etc/hosts`
    - using the windows: edit the `C:\Windows\System32\drivers\etc\hosts`

```hosts
127.0.0.1 traefik.x.internal
```

## Run

```shell
docker compose up -d
# docker compose -p traefik up -d
# docker compose -f ./compose.yml -p traefik up -d
```

Access: <http://traefik.x.internal>
