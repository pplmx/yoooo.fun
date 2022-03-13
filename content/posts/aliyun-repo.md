---
title: Some repos in aliyun
date: 2020-04-18T20:02:12+08:00
lastmod: 2020-06-24T11:04:34+08:00
slug: 59632d3d4355a000dd7a19f3bc3c663a
draft: false
categories: [general]
tags: [repo, pip, centos, ubuntu]
keywords: pip source, rpm repo, centos, ubuntu,
description: Some aliyun repo sources
---

# aliyun repo

## pip Repo

 ```bash
# ====== linux ======
mkdir ~/.pip
cat > ~/.pip/pip.conf << EOF
[global]
index-url=https://mirrors.aliyun.com/pypi/simple/
extra-index-url=https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/

[list]
format=columns

EOF
# ====== windows ======
mkdir $APPDATA/pip
touch $APPDATA/pip/pip.ini
# or (This is a legacy configuration)
mkdir $HOME/pip
touch $HOME/pip/pip.ini
 ```

## centos8 Repo

```
[AppStream]
name=CentOS-$releasever - aliyun - AppStream
baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
gpgcheck=0
enabled=1

[BaseOS]
name=CentOS-$releasever - aliyun - Base
baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
gpgcheck=0
enabled=1

[extras]
name=CentOS-$releasever - aliyun - extras
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os/
gpgcheck=0
enabled=1

[centosplus]
name=CentOS-$releasever - aliyun - centosplus
baseurl=https://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/os/
gpgcheck=0
enabled=1

[HighAvailability]
name=CentOS-$releasever - aliyun - HighAvailability
baseurl=https://mirrors.aliyun.com/centos/$releasever/HighAvailability/$basearch/os/
gpgcheck=0
enabled=1

[PowerTools]
name=CentOS-$releasever - aliyun - PowerTools
baseurl=https://mirrors.aliyun.com/centos/$releasever/PowerTools/$basearch/os/
gpgcheck=0
enabled=1

[epel]
name=CentOS-$releasever - aliyun - epel
baseurl=https://mirrors.aliyun.com/epel/$releasever/Everything/$basearch/
gpgcheck=0
enabled=1
```

## ubuntu18.04 Repo

```
deb https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```
