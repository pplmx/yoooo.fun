---
categories:
    - linux
date: 2020-04-18T20:56:44Z
description: how to customize cloud image config
keywords: cloud image, image, reset password
lastmod: 2020-04-19T18:12:04Z
tags:
    - image
title: config in cloud image
---



# change cloud image default config

## install package

```bash
sudo yum install -y libguestfs-tools
# or
sudo yum install -y libguestfs-tools-c
```

## Set root password

```bash
virt-customize -a rhel-server-7.6.qcow2 --root-password password:StrongRootPassword
```

## Register System

```bash
virt-customize -a overcloud-full.qcow2 --run-command 'subscription-manager register --username=[username] --password=[password]'

virt-customize -a rhel-server-7.6.qcow2 --run-command 'subscription-manager attach --pool [subscription-pool]'
```

## Install Software packages inside an image

```bash
virt-customize -a rhel-server-7.6.qcow2 --install [vim,bash-completion,wget,curl,telnet,unzip]

virt-customize -a rhel-server-7.6.qcow2 --install net-tools
```

## Upload SSH public key

```bash
# set ssh-key for a user(The user must exist in image)
virt-customize -a rhel-server-7.6.qcow2  --ssh-inject root:file:./id_rsa.pub
# or
virt-customize -a rhel-server-7.6.qcow2 --run-command 'useradd mystic' \
	--ssh-inject mystic:file:~/.ssh/id_rsa.pub
```

## Uploading files

```bash
virt-customize -a rhel-server-7.6.qcow2 --upload rhsm.conf:/etc/rhsm/rhsm.conf

virt-customize -a rhel-server-7.6.qcow2 --upload yum.conf:/etc/yum.conf

virt-customize -a rhel-server-7.6.qcow2 --upload proxy.sh:/etc/profile.d/
```

> The format: `local_file_path:image_file_path`

## Set Timezone

```bash
virt-customize -a rhel-server-7.6.qcow2 --timezone "Asia/Shanghai"
```

## Relabel SELinux

```bash
virt-customize -a rhel-server-7.6.qcow2 --selinux-relabel
```



