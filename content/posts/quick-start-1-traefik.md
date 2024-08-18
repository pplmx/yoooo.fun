---
categories:
    - docker
date: 2023-04-13T17:25:36Z
description: how to configure custom domain for traefik dashboard
keywords: docker,traefik,proxy,custom domain
lastmod: 2024-08-18T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
title: 'Quick Start: Traefik Dashboard with Custom Domain'
---



# Quick Start: Traefik

## Preparation

Create the necessary directories and files:

```shell
mkdir -p traefik/dynamic-conf && cd traefik && touch compose.yml traefik.yml dynamic-conf/self.yml
```

## Configuration Files

### compose.yml

```yaml
services:
    traefik:
        image: traefik:3.1
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

> Note: Mounting the Docker socket (`/var/run/docker.sock`) can pose security risks. Consider using more secure alternatives in production environments.

### traefik.yml

```yaml
### Static Configuration
log:
    level: INFO
api:
    insecure: true  # Warning: Not recommended for production use
    dashboard: true
entryPoints:
    web:
        address: :80
providers:
    file:
        directory: /etc/traefik/dynamic-conf
        watch: true
```

> Security Warning: `insecure: true` is not recommended for production environments. Consider setting up proper authentication for the API and dashboard.

### self.yml in dir dynamic-conf

```yaml
### Dynamic Configuration
http:
    routers:
        dashboard:
            rule: Host(`traefik.x.internal`)
            service: api@internal

```

## DNS Configuration

Configure your DNS or modify your hosts file:

- For Unix-like systems: Edit `/etc/hosts`
- For Windows: Edit `C:\Windows\System32\drivers\etc\hosts`

Add the following line:

```
127.0.0.1 traefik.x.internal
```

## Run

```shell
docker compose up -d
# Alternative commands:
# docker compose -p traefik up -d
# docker compose -f ./compose.yml -p traefik up -d
```

Access: http://traefik.x.internal
