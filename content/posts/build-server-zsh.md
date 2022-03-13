---
title: linux服务器初建之zsh安装
date: 2017-10-13T11:10:44+08:00
lastmod: 2021-11-30T10:20:00+08:00
slug: 70b23e561300acc0e2d6419494ef264f
draft: false
categories: [linux]
tags: [linux]
keywords: zsh, linux
description: install zsh on linux
---
# 安装zsh
    root下操作
## 安装zsh
```bash
yum -y install zsh
```
## 切换bash至zsh
```bash
chsh -s $(which zsh) $(whoami)
reboot
```
<!-- more -->

## install oh-my-zsh

```shell
# if wget
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# if curl
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# if your network is not fine, you can use the proxy
sh -c "$(curl -fsSLx http://www-proxy.example.com:8080 https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## config

```bash
vim ~/.zshrc
plugins=(git z zsh-autosuggestions zsh-syntax-highlighting)
ZSH_THEME="powerlevel10k/powerlevel10k"
POWERLEVEL10K_MODE="nerdfont-complete"
# you can run the command 'p10k configure' to reconfigure your powerlevel10k

git clone https://github.com/zsh-users/zsh-autosuggestions.git \
	${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
	${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/romkatv/powerlevel10k.git \
	${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

```

## fonts

You can download [Meslo font from here](https://github.com/ryanoasis/nerd-fonts).

Or Manually Download these four ttf files:
- [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)

- [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)

- [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)

- [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

Double-click on each file and click "Install". This will make `MesloLGS NF` font available to all applications on your system.

## BTW

**Tips**:

[fd](https://github.com/sharkdp/fd) [bat](https://github.com/sharkdp/bat) and [lsd](https://github.com/Peltoche/lsd) are very nice. 
Maybe you can install them. For more information, you can access their home page.

```bash
# install fd, bat and lsd on RHEL/CentOS
# Author="sharkdp"; Repo="fd";
# Author="sharkdp"; Repo="bat";
Author="Peltoche"; Repo="lsd"; latest_url="https://api.github.com/repos/$Author/$Repo/releases/latest"; \
    V=$(curl --silent $latest_url | grep -Eo '"tag_name": "v?(.*)"' | sed -E 's/.*"([^"]+)".*/\1/'); \
    Latest_tar="$Repo-$V-x86_64-unknown-linux-musl.tar.gz" \
    && curl -sOL "https://github.com/$Author/$Repo/releases/download/$V/$Latest_tar" \
    && tar xzvf $Latest_tar -C . \
    && sudo sh -c "cp ./$Repo-$V-x86_64-unknown-linux-musl/$Repo /usr/local/bin/$Repo" \
    && rm $Latest_tar
```