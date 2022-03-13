---
title: Some Info about SUSE
date: 2020-05-09T17:15:26+08:00
slug: 30f4f2f599735c36b84b8f3ad0eef3e2
draft: false
lastmod: 2020-05-09T17:20:58+08:00
categories: [linux]
tags: [opensuse]
keywords: opensuse, zypper
description: Some Info about SUSE
---
# zypper

```bash
# update all packages
sudo zypper refresh
# or sudo zypper ref
sudo zypper update
# or sudo zypper up

# list repos
zypper lr
# add a repo
zypper ar https://mirrors.aliyun.com/opensuse/update/leap/15.2/oss/ aliyun-suse-oss
# remove a repo
zypper rr aliyun-suse-oss

# enable the first repo
zypper mr -e 1
# disable the second repo
zypper mr -d 2
# enable caching for all repos
zypper mr -ka
# disable caching for all repos
zypper mr -Ka
# disable gpg check for all repos
zypper mr -Ga
```

## zypper some directories info

```text
/etc/zypp/zypper.conf
/etc/zypp/locks

# Directory containing repository definition (*.repo) files.
/etc/zypp/repos.d

# Directory containing service definition (*.service) files.
/etc/zypp/services.d

# System directory containing zypper extensions
/usr/lib/zypper/commands

# Directory for storing raw metadata contained in repositories.
/var/cache/zypp/raw

# Directory containing preparsed metadata in form of solv files.
/var/cache/zypp/solv

# If keeppackages property is set for a repository (see the modifyrepo command)
# all the RPM file downloaded during installation will be kept here.
/var/cache/zypp/packages

# Installation history log.
/var/log/zypp/history
```
