---
categories:
    - linux
date: 2021-03-18T11:12:08Z
description: By PPA, to install the latest
keywords: apt, apt-get, git, jdk, gradle
tags:
    - ubuntu
title: Install the Latest Packages by Ubuntu PPA
---



# Ubuntu

## Use PPA source(Latest Version)

### Git

```bash
# after this, it will generate a new list in /etc/apt/sources.list.d
sudo add-apt-repository ppa:git-core/ppa

# change "http://ppa.launchpad.net" to "https://launchpad.proxy.ustclug.org"
❯ ll /etc/apt/sources.list.d
.rw-r--r-- root root 137 B Fri Feb  5 17:49:07 2021 git-core-ubuntu-ppa-focal.list
❯ cat /etc/apt/sources.list.d/git-core-ubuntu-ppa-focal.list
deb https://launchpad.proxy.ustclug.org/git-core/ppa/ubuntu focal main

# install the latest git
sudo apt update
sudo apt install git
```

### Gradle

```bash
sudo add-apt-repository ppa:cwchien/gradle
```

### OpenJDK

```bash
sudo add-apt-repository ppa:openjdk-r/ppa
```

### Python

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
```

## Change the Default PPA Source

```bash
sed -i "s@http://ppa.launchpad.net@https://launchpad.proxy.ustclug.org@g" /etc/apt/sources.list.d/*.list

sed -i "s@http://mirrors.aliyun.com@https://mirrors.aliyun.com@g" /etc/apt/sources.list
sed -i "s@http://archive.ubuntu.com@https://mirrors.aliyun.com@g" /etc/apt/sources.list
```
