---
categories:
    - linux
date: 2017-10-13T11:10:44Z
description: install zsh on linux
keywords: zsh, linux
lastmod: 2024-01-13T11:13:20Z
tags:
    - linux
title: linux服务器初建之zsh安装
---



# zsh

## install

```bash
sudo yum install -y zsh

echo $(which zsh) | sudo tee -a /etc/shells
chsh -s $(which zsh) $(whoami)
```

## install oh-my-zsh

```shell
# export proxy if needed
export all_proxy=http://localhost:7890

# wget
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# or curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

```

## install some useful tools

### install by cargo

If rust is installed, you can use `cargo install` to install some tools.
If not, follow this [link](https://www.rust-lang.org/tools/install) to install rust.

```bash
cargo install ripgrep fd-find bat lsd
```

### install fzf

```bash
git clone https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

## config

```bash
vim ~/.zshrc
plugins=(git z sudo ripgrep fd fzf zsh-autosuggestions zsh-syntax-highlighting)
ZSH_THEME="powerlevel10k/powerlevel10k"
# you can run the command 'p10k configure' to reconfigure your powerlevel10k

git clone https://github.com/zsh-users/zsh-autosuggestions \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/romkatv/powerlevel10k \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

```
