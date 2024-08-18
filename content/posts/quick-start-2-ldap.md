---
categories:
    - docker
date: 2023-04-13T23:45:36Z
description: how to configure ldap based on traefik
keywords: docker,traefik,proxy,ldap,user,https
lastmod: 2023-11-24T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
    - ldap
    - login
    - user
title: 'Quick Start: LDAP'
---



# Quick Start: LDAP

If you want to use `bitnamic/openldap`,
please follow this [Quick Start: LDAP by Bitnami](https://blog.caoyu.info/quick-start-2_1-bitnami-ldap.html).

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

```yaml
services:
    ldap:
        image: osixia/openldap
        restart: always
        environment:
            - LDAP_ORGANISATION=Chaos Inc.
            # if LDAP_DOMAIN=chaos.io, the login DN will be "cn=admin,dc=chaos,dc=io"
            # LDAP_DOMAIN default value is "example.org"
            # so default login DN is "cn=admin,dc=example,dc=org"
            - LDAP_DOMAIN=chaos.io
            - LDAP_ADMIN_PASSWORD=secret
        volumes:
            - ldap:/var/lib/ldap
            - slapd:/etc/ldap/slapd.d
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
    ldap:
    slapd:

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
echo "127.0.0.1 ldap.x.internal\n" >> /etc/hosts
```

## Run

```shell
docker compose up -d
# docker compose -p ldap up -d
# docker compose -f ./compose.yml -p ldap up -d
```

Access: <https://ldap.x.internal>
