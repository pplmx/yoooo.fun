---
categories:
    - linux
date: 2024-01-13T11:18:20Z
description: Bootstrap a CentOS7 server with some necessary packages
keywords: zsh, linux, centos7
tags:
    - linux
    - centos7
title: Bootstrap a CentOS 7 server
---



# bootstrap a CentOS 7 server

## install gcc 11

```bash
yum install -y centos-release-scl
yum install -y devtoolset-11

# enable gcc 11 temporarily
scl enable devtoolset-11 bash
scl enable devtoolset-11 zsh

# enable gcc 11 permanently
echo "[ -f /opt/rh/devtoolset-11/enable ] && source /opt/rh/devtoolset-11/enable" >> ~/.bashrc
echo "[ -f /opt/rh/devtoolset-11/enable ] && source /opt/rh/devtoolset-11/enable" >> ~/.zshrc
```

## install zsh

```bash
sudo yum groupinstall "Development tools"
# sudo yum install -y gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel
sudo yum install -y ncurses-devel
wget --no-check-certificate https://www.zsh.org/pub/zsh-5.9.tar.xz
tar -xf zsh-5.9.tar.xz
cd zsh-5.9
./configure
make
sudo make install
echo "/usr/local/bin/zsh" | sudo tee -a /etc/shells

chsh -s /usr/local/bin/zsh
```

## install git

```bash
# check the old git version
git --version

# remove the old git
sudo yum -y remove git
sudo yum -y remove git-*

# install a repo
sudo yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum install git

# check git version
git --version
```

## install oh-my-zsh

Or follow this [link](https://blog.caoyu.info/build-server-zsh.html)

```bash
# configure proxy if needed
export all_proxy=http://localhost:7890

# install oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```


## install python from source

```bash
# download and extract
# PYTHON_VERSION=3.11.7
PYTHON_VERSION=3.8.18
wget -O /var/tmp/python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tar.xz"
install -d /var/tmp/python; tar -xf /var/tmp/python.tar.xz -C /var/tmp/python --strip-components 1

# compile
cd /var/tmp/python
./configure --enable-optimizations
make -j 8
# altinstall will not replace the system python,
# which means you need to use python3.8 and pip3.8 instead of python3 and pip3
sudo make altinstall

# of course, you can use ln -s to make python3.8 as default python3,
# because the centos 7 system python is 2.7
sudo ln -s /usr/local/bin/python3.8 /usr/local/bin/python3
sudo ln -s /usr/local/bin/pip3.8 /usr/local/bin/pip3
```

## install node

Node 17 is the latest version which supports CentOS 7, due to the `glibc` version.

```bash
# install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# install node
nvm ls-remote --lts
nvm install lts/gallium

# check node version
node -v

# config npm mirror
npm config set registry https://registry.npmmirror.com

# or config .npmrc manually
cat << EOF > ~/.npmrc
home="https://npmmirror.com"
registry="https://registry.npmmirror.com/"
electron_mirror="https://npmmirror.com/mirrors/electron/"
electron_custom_dir="{{ version }}"
electron_builder_binaries_mirror="http://npmmirror.com/mirrors/electron-builder-binaries/"

EOF
```

