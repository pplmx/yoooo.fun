---
categories:
    - linux
date: 2024-01-13T11:18:20Z
description: Bootstrap a CentOS7 server with some necessary packages
keywords: zsh, linux, centos7
lastmod: 2024-02-28T21:43:20Z
tags:
    - linux
    - centos7
title: Bootstrap a CentOS 7 server
---



# bootstrap a CentOS 7 server

## install gcc 11

```bash
sudo yum install -y centos-release-scl
sudo yum install -y devtoolset-11

# enable gcc 11 temporarily
scl enable devtoolset-11 bash
scl enable devtoolset-11 zsh

# enable gcc 11 permanently
echo "[ -f /opt/rh/devtoolset-11/enable ] && source /opt/rh/devtoolset-11/enable" >> ~/.bashrc
echo "[ -f /opt/rh/devtoolset-11/enable ] && source /opt/rh/devtoolset-11/enable" >> ~/.zshrc

# check the version
gcc --version
g++ --version
```

## install zsh

```bash
sudo yum groupinstall -y "Development tools"
sudo yum install -y gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel ncurses-devel
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
sudo yum remove -y git
sudo yum remove -y git-*

# install a repo
sudo yum install -y https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum install -y git

# check git version
git --version
```

## install oh-my-zsh

Follow this [link](https://blog.caoyu.info/build-server-zsh.html)

## install cmake

```bash
# download and extract
CMAKE_VERSION=3.28.3
wget https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}.tar.gz
tar -xf cmake-${CMAKE_VERSION}.tar.gz

# compile
cd cmake-${CMAKE_VERSION}
./configure
make -j 8
sudo make install

# check cmake version
cmake --version

# remove cmake source code
cd .. && rm -rf cmake-${CMAKE_VERSION}*
```

### install docker

FYI: [install docker](https://docs.docker.com/engine/install/centos/)

```bash
sudo yum install -y docker-buildx-plugin
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# start docker
sudo systemctl enable docker
sudo systemctl start docker

# post installation
sudo /usr/sbin/groupadd docker
sudo /usr/sbin/usermod -aG docker $USER
newgrp docker
```

### install java

```bash
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm
sudo yum install -y ./jdk-17_linux-x64_bin.rpm
rm -f jdk-17_linux-x64_bin.rpm

```

### install go

Follow this [link](https://blog.caoyu.info/install-binary-file.html)

## install python from source

```bash
# download and extract
# PYTHON_VERSION=3.12.2
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

### install nvim

```bash
NVIM_VERSION=0.9.5
wget https://github.com/neovim/neovim/releases/download/v${NVIM_VERSION}/nvim.appimage

chmod u+x nvim.appimage && ./nvim.appimage

# if no error, move it to /usr/local/bin
sudo mv nvim.appimage /usr/local/bin/nvim

# if error
./nvim.appimage --appimage-extract
./squashfs-root/usr/bin/nvim
sudo mv squashfs-root/usr/bin/nvim /usr/local/bin/nvim

# check nvim version
nvim --version
```

### install LunarVim

```bash
LV_BRANCH='release-1.3/neovim-0.9' bash <(curl -s https://raw.githubusercontent.com/LunarVim/LunarVim/release-1.3/neovim-0.9/utils/installer/install.sh)

# add the following to ~/.config/lvim/config.lua
vim.opt.shiftwidth = 4 -- the number of spaces inserted for each indentation
vim.opt.tabstop = 4 -- insert spaces for a tab
```
