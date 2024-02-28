---
categories:
    - linux
date: 2024-02-28T21:47:20Z
description: How to use node 18/20 on CentOS 7
keywords: node, 18, 20, glibc, linux, centos7, libstdc++
tags:
    - linux
    - centos7
title: Use node 18/20 on CentOS 7
---



# how to use node 18/20 on centos7

## install node by nvm

```bash
# install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# install node
nvm ls-remote --lts

# node 16 is lts/gallium, node 20 is lts/iron
nvm install lts/gallium
nvm install lts/iron

# check node version
nvm alias default lts/gallium
node -v

nvm alias default lts/iron
node -v
```

You will find that node 16 works well, but node 20 will not work, because the `glibc` version is too low.
That's so bad, but don't worry; we can fix it by the following steps.

## install glibc 2.31 and libstdc++.so.6.0.25

Follow my another article [centos7-upgrade-libc](https://blog.caoyu.info/centos7-upgrade-libc.html) to upgrade `glibc` to 2.31.

## test

```bash
nvm alias default lts/iron
node -v # it works well
```
