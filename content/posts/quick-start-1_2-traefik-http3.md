---
categories:
    - docker
date: 2023-04-16T12:18:36Z
description: how to enable http3 for traefik
keywords: HTTP/3,http3,SSL,HTTPS,docker,traefik,openssl,certificates,proxy,custom domain,self-signed certificate
lastmod: 2024-08-18T10:42:36Z
tags:
    - http3
    - https
    - ssl
    - docker
    - traefik
    - openssl
    - certificates
    - proxy
title: 'Quick Start: Traefik with HTTP/3'
---



# Quick Start: Traefik with HTTP/3

## Preparation

Create the necessary directories and files:

```shell
mkdir -p traefik/dynamic-conf traefik/certs && cd traefik && touch compose.yml traefik.yml dynamic-conf/self.yml
```

## Configuration Files

### compose.yml

```yaml
services:
    traefik:
        image: traefik:3.1
        ports:
            - "80:80"
            - "443:443/tcp"
            - "443:443/udp"
        environment:
            - TZ=Asia/Shanghai
        volumes:
            # /traefik.yml and /etc/traefik/traefik.yml are both available.
            - "./traefik.yml:/etc/traefik/traefik.yml"
            # dynamic-conf dir is self-defined
            - "./dynamic-conf:/etc/traefik/dynamic-conf"
            - "./certs:/certs"
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
    dashboard: true
entryPoints:
    web:
        address: :80
        http:
            redirections:
                entryPoint:
                    to: websecure
                    scheme: https
                    permanent: true
    websecure:
        address: :443
        http3: { }
providers:
    file:
        directory: /etc/traefik/dynamic-conf
        watch: true

```

### self.yml in dir dynamic-conf

```yaml
### Dynamic Configuration
tls:
    certificates:
        -   certFile: /certs/cert.pem
            keyFile: /certs/key.pem
http:
    routers:
        dashboard:
            rule: Host(`traefik.x.internal`)
            service: api@internal
            tls: { }

```

## DNS Configuration

Configure your DNS or modify your hosts file:

- For Unix-like systems: Edit `/etc/hosts`
- For Windows: Edit `C:\Windows\System32\drivers\etc\hosts`

Add the following line:

```
127.0.0.1 traefik.x.internal
```

## Generate Self-Signed Certificates

### Option-1: Using mkcert

`mkcert` installation is here: https://github.com/FiloSottile/mkcert

`mkcert` can solve the problem of browser distrust
If you want to solve this problem, then `mkcert` is the best choice.

```shell
# directly gen certs at the current dir
# mkcert example.com "*.example.com" example.test localhost 127.0.0.1 ::1

# specify the cert output dir
mkcert -key-file certs/key.pem -cert-file certs/cert.pem x.internal "*.x.internal"
mkcert -install
```

### Option-2: Using openssl

- **option-a**: configure with command line

```shell
openssl req -new -x509 -nodes -newkey rsa:4096 -days 365 \
    -subj "/C=CN/ST=SH/L=Shanghai/CN=*.x.internal" \
    -keyout certs/key.pem \
    -out certs/cert.pem
```

- **option-b**: configure with a `ssl.cnf`

```shell
# When using -x509, default_days in config will be ignored, it is a bug
# using -days to workaround
openssl req -x509 -new -nodes -days 365 \
    -config ssl.cnf \
    -keyout certs/key.pem \
    -out certs/cert.pem
```

`ssl.cnf` like as follows:

Tips: `DNS.1`, `DNS.2`, `IP.7`, `DNS.11`, the numbers are only required to be unique, and can also be unordered.

```shell
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
x509_extensions    = v3_req

[ req_distinguished_name ]
C  = CN
ST = SH
L  = Shanghai
O  = Individual
OU = MyStudio
CN = x.internal

[ v3_req ]
subjectAltName = @alt_names

[alt_names]
DNS.1  = x.internal
DNS.2  = *.x.internal
IP.7   = 127.0.0.1
DNS.11 = localhost

```

## Run

```shell
docker compose up -d
# Alternative commands:
# docker compose -p traefik up -d
# docker compose -f ./compose.yml -p traefik up -d
```

Access: https://traefik.x.internal
