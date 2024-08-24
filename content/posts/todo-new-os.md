---
categories:
    - general
date: 2023-07-28T10:40:11Z
description: install some necessary tools for me on a new OS
keywords: todo,tools,preparation,setup,pipx,cargo,scoop,winget
lastmod: 2024-08-24T10:42:36Z
tags:
    - todo
    - tools
title: TODO on a new OS
---



# New OS Setup Guide

## General Setup

### UV Tools

To install essential development tools using UV, follow these steps:

1. [Install UV](https://docs.astral.sh/uv/getting-started/installation/)
2. Install necessary tools:

   ```shell
   uv tool install commitizen
   uv tool install cookiecutter
   uv tool install hatch
   uv tool install pre-commit
   uv tool install ruff
   ```

3. Verify the installation:

   ```shell
   uv tool dir --bin
   uv tool list
   ```

### Cargo Tools

Install some common Rust tools:

```shell
cargo install zoxide bat fd-find ripgrep
```

> **Note:** If you encounter any errors, try using the `--locked` option:

```shell
cargo install --locked bat
```

## Windows Setup

### Scoop

Install essential and optional tools using Scoop:

1. **Essential Tools:**

   ```shell
   scoop install 7zip curl fzf gsudo Meslo-NF Meslo-NF-Mono neovim scoop-search
   ```

2. **Development Tools:**

   ```shell
   scoop install buf ccache cmake gcc gradle make maven mkcert sccache xmake
   ```

3. **Additional Tools:**

   ```shell
   scoop install hugo-extended
   ```

### Winget

Install `Oh My Posh` using Winget:

```shell
winget install JanDeDobbeleer.OhMyPosh
```

## Linux Setup

### RHEL (Red Hat Enterprise Linux)

For development and virtualization setups:

1. **Development Libraries and Tools:**

   ```shell
   sudo dnf install -y @development-libs
   sudo dnf install -y @development-tools
   ```

2. **Virtualization Setup (QEMU and KVM):**

   ```shell
   sudo dnf install -y @virtualization
   ```

   Enable and start the virtualization service:

   ```shell
   sudo systemctl enable --now libvirtd
   sudo systemctl start libvirtd
   ```

### Debian/Ubuntu

Install build essentials for development:

```shell
sudo apt install -y build-essential
```
