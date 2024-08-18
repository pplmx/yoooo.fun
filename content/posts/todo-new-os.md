---
categories:
    - general
date: 2023-07-28T10:40:11Z
description: install some necessary tools for me on a new OS
keywords: todo,tools,preparation,setup,pipx,cargo,scoop,winget
lastmod: 2024-01-06T22:21:46Z
tags:
    - todo
    - tools
title: TODO on a new OS
---



# Some Preparation for me on a New OS

## General

### pipx

```shell
pipx install commitizen pre-commit cookiecutter ruff uv hatch
```

### cargo

```shell
cargo install zoxide bat fd-find ripgrep

# if error, try to locked, e.g.
cargo install --locked bat
```

## Windows

### scoop

```shell
scoop install 7zip btop cmake curl fzf gcc gradle gsudo make Meslo-NF Meslo-NF-Mono neovim scoop-search
```

### winget

```shell
winget install Git.Git
winget install appmakes.Typora
winget install OpenJS.NodeJS
winget install GoLang.Go
winget install JanDeDobbeleer.OhMyPosh
```

## Linux

### REHL

```shell
# for development
sudo dnf install -y @development-libs
sudo dnf install -y @development-tools

# for virtualization, such as qemu and kvm
sudo dnf install -y @virtualization
# after installation, enable libvirtd
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
```

### Debian/Ubuntu

```shell
sudo apt install -y build-essential
```
