---
categories:
    - linux
date: 2024-02-28T21:45:20Z
description: Upgrade glibc from 2.17 to 2.31 on CentOS 7
keywords: glibc, linux, centos7, libstdc++
tags:
    - linux
    - centos7
title: How to upgrade glibc on CentOS 7
---



# how to upgrade glibc on centos7

## check the current version

```bash
gcc --version
g++ --version

locate libc.so.6
locate libstdc++.so.6
strings /usr/lib64/libc.so.6 | grep -E "^GLIBC_"
strings /usr/lib64/libstdc++.so.6 | grep -E "^GLIBCXX_"
```

## install gcc 8

```bash
sudo yum install -y centos-release-scl
sudo yum install -y devtoolset-8

# enable gcc 11 temporarily
scl enable devtoolset-8 bash
scl enable devtoolset-8 zsh

# enable gcc 11 permanently
echo "[ -f /opt/rh/devtoolset-8/enable ] && source /opt/rh/devtoolset-8/enable" >> ~/.bashrc
echo "[ -f /opt/rh/devtoolset-8/enable ] && source /opt/rh/devtoolset-8/enable" >> ~/.zshrc

# check the version
gcc --version
g++ --version
```

## install glibc 2.31

```bash
sudo yum groupinstall -y  "Development tools"
sudo yum install -y gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel ncurses-devel nss-devel

GLIBC_VERSION="2.31"
# GNU="https://ftp.gnu.org/gnu"
GNU_MIRROR="https://mirrors.aliyun.com/gnu"
wget ${GNU_MIRROR}/glibc/glibc-${GLIBC_VERSION}.tar.xz
tar -xf glibc-${GLIBC_VERSION}.tar.xz && cd glibc-${GLIBC_VERSION}

mkdir build; cd build
../configure --prefix=/usr --with-headers=/usr/include --with-binutils=/usr/bin --disable-profile --enable-add-ons
../configure --prefix=/usr \
    --with-headers=/usr/include \
    --with-binutils=/usr/bin \
    --enable-add-ons \
    --enable-obsolete-nsl \
    --disable-profile \
    --disable-sanity-checks \
    --disable-werror
make -j8
sudo make install
```

### issue: "ld: cannot find -lnss_test2"

Maybe you will encounter the following error:

> /usr/bin/ld: cannot find -lnss_test2

You can fix it by the following [this](https://garlicspace.com/2020/07/18/centos7-%E5%8D%87%E7%BA%A7-glibc-gcc/#make_install):

> vim ../scripts/test-installation.pl +128
>
> Before changed:
>
>         && $name ne "nss_test1" && $name ne "libgcc_s") {
>
> append this condition `&& $name ne "nss_test2"` to skip the test of nss_test2
>
> Changed:
>
>         && $name ne "nss_test1" && $name ne "nss_test2" && $name ne "libgcc_s") {
>

## upgrade libstdc++, if necessary

After you have installed glibc 2.31, you also replace the old `libstdc++.so.6` with the new one.

Before replacing, it looks like this:

`ls -l /usr/lib64/libstdc++.so.6*`

```log
lrwxrwxrwx. 1 root root   19 Feb 27 22:24 /usr/lib64/libstdc++.so.6 -> libstdc++.so.6.0.19
-rwxr-xr-x. 1 root root 973K Sep 29  2020 /usr/lib64/libstdc++.so.6.0.19
```

We will replace it with `libstdc++.so.6.0.25`.
But how to get it? A simple way is to use docker image to get it, like this:

```bash
# show the libc and libstdc++ version
❯ docker container run -it --rm centos:8 bash -c "ls -l /lib64/libc.so.6*"
lrwxrwxrwx 1 root root 12 Mar 11  2021 /lib64/libc.so.6 -> libc-2.28.so
❯ docker container run -it --rm centos:8 bash -c "ls -l /lib64/libstdc++.so.6*"
lrwxrwxrwx 1 root root      19 Oct 12  2020 /lib64/libstdc++.so.6 -> libstdc++.so.6.0.25
-rwxr-xr-x 1 root root 1661392 Oct 12  2020 /lib64/libstdc++.so.6.0.25

# start a container and copy libstdc++ to the host
docker container run -d --name t1 centos:8 init
docker container cp t1:/usr/lib64/libstdc++.so.6.0.25 /var/tmp

# replace the host libstdc++
sudo cp /var/tmp/libstdc++.so.6.0.25 /usr/lib64
sudo rm -fr /usr/lib64/libstdc++.so.6
cd /usr/lib64; sudo ln -s libstdc++.so.6.0.25 libstdc++.so.6
```

Replaced, it looks like this:

`ls -l /usr/lib64/libstdc++.so.6*`

```log
lrwxrwxrwx. 1 root root      19 Feb 27 22:24 /usr/lib64/libstdc++.so.6 -> libstdc++.so.6.0.25
-rwxr-xr-x. 1 root root  995840 Sep 29  2020 /usr/lib64/libstdc++.so.6.0.19
-rwxr-xr-x. 1 root root 1661392 Feb 27 22:24 /usr/lib64/libstdc++.so.6.0.25
```

## Question

### gnome-terminal cannot open

try this command: `sudo localedef -f UTF-8 -i en_US en_US.UTF-8`
