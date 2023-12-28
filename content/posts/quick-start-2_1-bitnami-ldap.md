---
categories:
    - docker
date: 2023-04-15T23:16:36Z
description: how to configure bitnami openldap based on traefik
keywords: docker,traefik,proxy,ldap,bitnami,openldap,user,https
lastmod: 2023-11-24T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
    - ldap
    - login
    - user
title: 'Quick Start: LDAP by Bitnami'
---



# Quick Start: LDAP by Bitnami

## Prerequisite

> [Traefik on HTTP](https://blog.caoyu.info/quick-start-1-traefik.html)
>
> OR
>
> [Traefik on HTTPS](https://blog.caoyu.info/quick-start-1-1-traefik-ssl.html)
>
> If HTTP, remove the `tls: {}` in dynamic configuration

## Preparation

### compose.yml

> For much more information, please reference [here](https://hub.docker.com/r/bitnami/openldap/).

```yaml
version: Compose specification

services:
    ldap:
        image: bitnami/openldap
        restart: always
        environment:
            - LDAP_ADMIN_USERNAME=admin
            - LDAP_ADMIN_PASSWORD=secret
            - LDAP_ROOT=dc=chaos,dc=io
            # for using phpldapadmin, config 389 and 636
            # if not, remove the following two environments
            - LDAP_PORT_NUMBER=389
            - LDAP_LDAPS_PORT_NUMBER=636
        volumes:
            - openldap:/bitnami/openldap
        networks:
            - traefik-net

    ldapadmin:
        image: osixia/phpldapadmin
        restart: always
        environment:
            - PHPLDAPADMIN_LDAP_HOSTS=ldap
            # if configure https by traefik, you need to configure the following two lines
            # if not, remove them
            - VIRTUAL_HOST=ldap.x.internal
            - PHPLDAPADMIN_HTTPS=false
        networks:
            - traefik-net
volumes:
    openldap:

networks:
    traefik-net:
        external: true

```

### ldap.yml in dir dynamic-conf

> You should touch `ldap.yml` in traefik dir **dynamic-conf**.
>
> For much more information, please reference the [Prerequisite](#Prerequisite).

```yaml
http:
    routers:
        ldap:
            rule: "Host(`ldap.x.internal`)"
            service: "ldap"
            tls: { }

    services:
        ldap:
            loadBalancer:
                servers:
                    -   url: "http://ldapadmin"

```

## Config domain parse

```shell
echo "127.0.0.1 bitnami-ldap.x.internal\n" >> /etc/hosts
```

## Run

```shell
docker compose up -d
# docker compose -p bitnami-ldap up -d
# docker compose -f ./compose.yml -p bitnami-ldap up -d
```

Access: <https://bitnami-ldap.x.internal>
