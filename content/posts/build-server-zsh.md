---
categories:
    - linux
date: 2017-10-13T11:10:44Z
description: install zsh on linux
keywords: zsh, linux
lastmod: 2024-08-18T10:42:36Z
tags:
    - linux
title: linux服务器初建之zsh安装
---



# Oh My Zsh Installation Guide

## 1. Install Zsh

First, install the Zsh shell:

```bash
# RedHat/Fedora/CentOS
sudo yum install -y zsh

# Debian/Ubuntu
sudo apt-get update
sudo apt-get install -y zsh

# macOS (using Homebrew)
brew install zsh
```

Verify the installation:

```bash
zsh --version
```

## 2. Set Zsh as Default Shell

```bash
# Add Zsh to available shells
command -v zsh | sudo tee -a /etc/shells

# Change default shell
chsh -s $(command -v zsh)
```

Note: You'll need to log out and log back in for the change to take effect.

## 3. Install Oh My Zsh

If you need to use a proxy, set the proxy environment variable first:

```bash
# Modify proxy address and port according to your setup
export all_proxy=http://localhost:7890
```

Choose one of the following installation methods:

```bash
# Install using curl (recommended)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Or install using wget
sh -c "$(wget -qO- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

```

## 4. Install Recommended Tools

### 4.1 Install Modern CLI Tools via Rust/Cargo

First, install Rust (if not already installed):

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

Install commonly used tools:

```bash
# ripgrep: faster alternative to grep
# fd-find: better alternative to find
# bat: better alternative to cat with syntax highlighting
# lsd: modern alternative to ls with icons
cargo install ripgrep fd-find bat lsd
```

### 4.2 Install fzf (Fuzzy Finder)

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

## 5. Configure Oh My Zsh

### 5.1 Install Recommended Plugins

```bash
# Auto-suggestions plugin
git clone --depth 1 https://github.com/zsh-users/zsh-autosuggestions \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Syntax highlighting plugin
git clone --depth 1 https://github.com/zsh-users/zsh-syntax-highlighting \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# PowerLevel10k theme (optional but recommended)
git clone --depth 1 https://github.com/romkatv/powerlevel10k \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

### 5.2 Edit Configuration

Edit your `~/.zshrc` file:

```bash
# Set theme (if PowerLevel10k is installed)
ZSH_THEME="powerlevel10k/powerlevel10k"

# Enable plugins
plugins=(
    git             # Git integration
    z               # Quick directory jumping
    sudo            # Double ESC to add sudo
    zsh-autosuggestions
    zsh-syntax-highlighting
)

# Optional: Set up useful aliases
alias ls='lsd'
alias cat='bat'
alias grep='rg'
alias find='fd'
```

### 5.3 Apply Changes

```bash
source ~/.zshrc
```

If using PowerLevel10k theme, the configuration wizard will start automatically on first load. You can also run it manually:

```bash
p10k configure
```

## Troubleshooting

1. If you encounter permission issues:

    ```bash
    sudo chown -R $USER:$USER ~/.oh-my-zsh
    ```

2. If plugin installation fails, check your network connection and proxy settings:

    ```bash
    # Set Git proxy
    git config --global http.proxy http://localhost:7890
    git config --global https.proxy http://localhost:7890
    ```

3. If theme looks incorrect, ensure recommended fonts are installed:

    - Install Nerd Font
    - Set your terminal emulator to use a Nerd Font
