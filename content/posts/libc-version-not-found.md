---
categories:
    - docker
date: 2023-07-28T10:48:32Z
description: use specific software version and specify system version
keywords: docker,multi-stage,libc,musl,libstdc++,version-not-found
lastmod: 2023-07-28T14:06:12Z
tags:
    - docker
    - multi-stage
    - libc
    - musl
    - libstdc++
title: How to handle libc version not found in docker multi-stage build
---



# About debugging the issue "glibc version not found"

## Pre

Recently, I wanna build an image from the `previous Dockerfile` (Which absolutely works fine before).

However, it stuck in `libc version not found` when I ran it.

## Dockerfile

Here is my Dockerfile

```dockerfile
FROM golang:1.20 AS so
# build dynamic lib
# ...

FROM node:18 AS env
# install node packages
# ...

FROM gcr.io/distroless/nodejs18-debian11
# running js app with golang dynamic lib
```

## 1st issue

Upon running the image, it throws:

> /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version GLIBCXX_3.4.29 not found(required by ffi-napi)

I realized that maybe the problem is in the second `node:18` image, so I changed it to `node:16`, but the problem still exists; I tried `node:20`
image again, still again.

After a lot of searching, I found that the `node:18` image was upgraded from the default system version `bullseye` to `bookworm`, so I tried to use
the `node:18-bullseye` image, and the problem was solved.

Then, I dived into the second issue. ><...

## 2nd issue

After I fixed the 1st issue, it threw another error:

> /lib/x86_64-linux-gnu/libc.so.6: version GLIBC_2.32 not found(required by my golang dynamic lib)

Due to the experience of the first issue, after confirming that the `golang` image has this version `golang:1.20-bullseye`, I immediately tried
the `golang:1.20-bullseye` image, but the problem was not solved as I expected. Sad.

After a long time of thinking, I suddenly realized that maybe the system version(Ubuntu 22) of my docker is too high?

I immediately tried to use my Ubuntu 18.04 system version to run, and the problem was solved! Wonderful!!!

BTW, Changing `golang:1.20` to `golang:1.20-bullseye` is **necessary**, if not, it still
throws `/lib/x86_64-linux-gnu/libc.so.6: version GLIBC_2.32 not found`

## Fixed Dockerfile

```dockerfile
FROM golang:1.20-bullseye AS so
# build dynamic lib
# ...

FROM node:18-bullseye AS env
# install node packages
# ...

FROM gcr.io/distroless/nodejs18-debian11
# running js app with golang dynamic lib
```

## Conclusion

- When using FROM in Dockerfile, not only need to specify the software version, but also need to specify the system version
- The host system version of Docker is not easy to control, just don't be too high or too low
