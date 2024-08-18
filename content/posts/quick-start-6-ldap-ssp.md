---
categories:
    - docker
date: 2023-07-12T15:20:21Z
description: how to configure ssp for ldap based on traefik
keywords: docker,traefik,proxy,ssp,password,https
lastmod: 2023-11-24T10:42:36Z
tags:
    - docker
    - traefik
    - proxy
    - ssp
    - password
title: 'Quick Start: SSP'
---



# Quick Start: SSP

SSP(`Self-Service Password`), is a tool for ldap to change password.

## Prerequisite

> - [Traefik on HTTP](https://blog.caoyu.info/quick-start-1-traefik.html)
>
> OR
>
> - [Traefik on HTTPS](https://blog.caoyu.info/quick-start-1-1-traefik-ssl.html)
>
> Note: If using HTTP, remove the `tls: {}` in dynamic configuration.
>
> - [LDAP](https://blog.caoyu.info/quick-start-2_1-bitnami-ldap.html)

## Preparation

### compose.yml

```yaml
services:
    ssp:
        image: ltbproject/self-service-password
        volumes:
            - ./ssp.conf.php:/var/www/conf/config.inc.local.php
        networks:
            - traefik-net

networks:
    traefik-net:
        external: true

```

### configuration

```php
<?php
// general
$keyphrase = "mysecret";
// $debug = true;
// $smarty_debug = true;
$login_forbidden_chars = "*()&|";

// ldap connection
// ldap-srv is your ldap service name in docker compose file
$ldap_url = "ldap://ldap-srv:1389";
$ldap_binddn = "cn=admin,dc=chaos,dc=io";
$ldap_bindpw = "secret";
$who_change_password = "manager";
$ldap_base = "ou=users,dc=chaos,dc=io";
$ldap_filter = "(&(objectClass=person)(uid={login}))";

// password policy
$hash = "auto";
$pwd_min_length = 12;
$pwd_max_length = 30;
$pwd_min_lower = 1;
$pwd_min_upper = 1;
$pwd_min_digit = 1;
$pwd_min_special = 1;
$pwd_special_chars = "^a-zA-Z0-9"; // This means special characters are all characters except alphabetical letters and digits.
$pwd_no_special_at_ends = true; // Special characters are not allowed at the beginning or at the end of the password.
$pwd_show_policy = "always"; // never, onerror, always
$pwd_show_policy_pos = "above"; // above, below
$show_extended_error = true;

// reset by mail tokens
$use_tokens = true;
$mail_address_use_ldap = true;

?>

```

### ssp.yml in dir dynamic-conf

> You should touch `ssp.yml` in traefik dir **dynamic-conf**.
>
> For Much more information, please reference the [Prerequisite](#Prerequisite).

```yaml
http:
    routers:
        ssp:
            rule: "Host(`ssp.x.internal`)"
            service: "ssp"
            tls: { }

    services:
        ssp:
            loadBalancer:
                servers:
                    -   url: "http://ssp"

```

## DNS Configuration

Configure your DNS or modify your hosts file:

- For Unix-like systems: Edit `/etc/hosts`
- For Windows: Edit `C:\Windows\System32\drivers\etc\hosts`

Add the following line:

```
127.0.0.1 ssp.x.internal
```

## Run

```shell
docker compose up -d
# Alternative commands:
# docker compose -p ssp up -d
# docker compose -f ./compose.yml -p ssp up -d
```

Access: https://ssp.x.internal

## FYI

<https://github.com/ltb-project/self-service-password>

<https://self-service-password.readthedocs.io/>
