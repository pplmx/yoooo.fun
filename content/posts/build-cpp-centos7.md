---
categories:
    - docker
date: 2023-03-10T16:58:34Z
description: build cpp project with gcc 11 on CentOS7 image
keywords: cpp,gcc11,centos7
lastmod: 2023-03-10T16:58:34Z
tags:
    - cpp
    - linux
    - centos7
    - gcc
    - docker
title: Build cpp with gcc11 on CentOS7
---



```dockerfile
### BUILDING
FROM centos:7 AS builder
LABEL author="Mystic"
ARG CMAKE_VERSION="3.24.2"

WORKDIR /var/tmp
RUN curl -LO https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz &&\
    tar -zxf cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    \cp -fr cmake-${CMAKE_VERSION}-linux-x86_64/* /usr/local
RUN yum upgrade -y
# install gcc 11
RUN yum install -y centos-release-scl &&  \
    yum install -y devtoolset-11

WORKDIR /app

# To ensure the symlink works fine on Windows, you must do the following:
# 1. `git config --global core.symlinks true`
# 2. Enable `Developer Mode` to authorize `mklink` permission
# 3. reclone your repo
COPY . .

WORKDIR build

# Run cmake with gcc 11
RUN scl enable devtoolset-11 'cmake .. && cmake --build . --target sample --config Release --parallel 8'

### DEPLOYING
FROM centos:7

COPY --from=builder /app/build/sample /sample
COPY test .

CMD ["/sample"]
```
