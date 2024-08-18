---
categories:
    - docker
date: 2023-04-14T14:14:21Z
description: how to configure jenkins with https based on traefik
keywords: docker,traefik,proxy,jenkins,ci,cd,https
lastmod: 2024-08-18T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
    - jenkins
    - ci
    - cd
title: 'Quick Start: Jenkins'
---



# Quick Start: Jenkins

## Prerequisite

> - [Traefik on HTTP](https://blog.caoyu.info/quick-start-1-traefik.html)
>
> OR
>
> - [Traefik on HTTPS](https://blog.caoyu.info/quick-start-1-1-traefik-ssl.html)
>
> Note: If using HTTP, remove the `tls: {}` in dynamic configuration.

## Preparation

### compose.yml

```yaml
services:
    jenkins:
        image: jenkins/jenkins
        container_name: jenkins
        privileged: true
        user: root
        expose:
            - 8080
            - 50000
        restart: always
        extra_hosts:
            # config for gerrit, sonar, ldap. if no, remove them
            - gerrit.x.internal:192.168.91.103
            - sonar.x.internal:192.168.91.103
            - ldap.x.internal:192.168.91.103
        volumes:
            - jenkins_home:/var/jenkins_home
            - /var/run/docker.sock:/var/run/docker.sock
        environment:
            - TZ=Asia/Shanghai
        networks:
            - traefik-net

volumes:
    jenkins_home:

networks:
    traefik-net:
        external: true

```

### jenkins.yml in dir dynamic-conf

> You should touch `jenkins.yml` in traefik dir **dynamic-conf**.
>
> For much more information, please reference the [Prerequisite](#Prerequisite).

```yaml
http:
    routers:
        jenkins:
            rule: "Host(`jenkins.x.internal`)"
            service: "jenkins"
            tls: { }

    services:
        jenkins:
            loadBalancer:
                servers:
                    -   url: "http://jenkins:8080"

```

## DNS Configuration

Configure your DNS or modify your hosts file:

- For Unix-like systems: Edit `/etc/hosts`
- For Windows: Edit `C:\Windows\System32\drivers\etc\hosts`

Add the following line:

```
127.0.0.1 jenkins.x.internal
```

## Run

```shell
docker compose up -d
# Alternative commands:
# docker compose -p jenkins up -d
# docker compose -f ./compose.yml -p jenkins up -d
```

Access: https://jenkins.x.internal
