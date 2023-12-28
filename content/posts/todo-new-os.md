---
categories:
    - notes
date: 2023-07-28T10:40:11Z
description: install some necessary tools for me on a new OS
keywords: todo,tools,preparation,setup,pipx,cargo,scoop,winget
lastmod: 2023-07-28T14:06:12Z
tags:
    - todo
    - tools
title: TODO on a new OS
---



# Some Preparation for me on a New OS

## General

### pipx

```shell
pipx install black commitizen flake8 hatch httpie pipenv poetry pre-commit
```

### cargo

```shell
cargo install cargo-generate zoxide bat fd-find ripgrep

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
winget install Fndroid.ClashForWindows
winget install appmakes.Typora
winget install OpenJS.NodeJS
winget install GoLang.Go
winget install NetEase.CloudMusic
winget install JanDeDobbeleer.OhMyPosh
```
