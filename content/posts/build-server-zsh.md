---
categories:
    - linux
date: 2017-10-13T11:10:44Z
description: install zsh on linux
keywords: zsh, linux
lastmod: 2025-01-18T10:42:36Z
tags:
    - linux
title: linux服务器初建之zsh安装
---



# Oh My Zsh Installation Guide

## 1. Core Installation

### 1.1 Install Zsh

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

### 1.2 Set Zsh as Default Shell

```bash
# Add Zsh to available shells
command -v zsh | sudo tee -a /etc/shells

# Change default shell
chsh -s $(command -v zsh)
```

Note: You'll need to log out and log back in for the change to take effect.

### 1.3 Install Oh My Zsh

Follow this step only if you need to use a proxy; otherwise, feel free to skip it.

```bash
# Set proxy for shell
export all_proxy=http://localhost:7890

# Set Git proxy if needed
git config --global http.proxy http://localhost:7890
git config --global https.proxy http://localhost:7890
```

Install Oh My Zsh:

```bash
# Install using curl (recommended)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Or install using wget
sh -c "$(wget -qO- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 1.4 Install Default Plugins

```bash
# Install auto-suggestions
git clone --depth 1 https://github.com/zsh-users/zsh-autosuggestions \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Install syntax highlighting
git clone --depth 1 https://github.com/zsh-users/zsh-syntax-highlighting \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Install fzf
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### 1.5 Basic Configuration

Edit your `~/.zshrc` file:

```bash
# Enable plugins
plugins=(
    git                     # Git integration
    z                       # Quick directory jumping
    sudo                    # Double ESC to add sudo
    zsh-autosuggestions     # Auto-suggestions
    zsh-syntax-highlighting # Syntax highlighting
)

# Apply changes
source ~/.zshrc
```

## 2. Optional Enhancements

The above is the minimal setup for me. For more advanced configurations, you can customize your `~/.zshrc` file further.

### 2.1 PowerLevel10k Theme

Install the theme:

```bash
git clone --depth 1 https://github.com/romkatv/powerlevel10k \
    ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Configure in `~/.zshrc`:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Run configuration wizard:

```bash
p10k configure
```

Required font setup:

- Install `Nerd Font`: https://github.com/ryanoasis/nerd-fonts/releases/latest
- Set your `terminal emulator` to use a `Nerd Font`

### 2.2 Modern CLI Tools (via Rust)

Install Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

Install enhanced command-line tools:

```bash
# ripgrep: faster alternative to grep
# fd-find: better alternative to find
# bat: better alternative to cat with syntax highlighting
# lsd: modern alternative to ls with icons
cargo install ripgrep fd-find bat lsd
```

Add to `~/.zshrc` if using Rust tools:

```bash
alias ls='lsd'
alias cat='bat'
alias grep='rg'
alias find='fd'
```

## 3. Troubleshooting

1. Permission issues:

    ```bash
    sudo chown -R $USER:$USER ~/.oh-my-zsh
    ```

2. Network-related issues:

    - Check your internet connection
    - Verify proxy settings if using a proxy
    - Try downloading files directly if git clone fails

3. Plugin issues:

    ```bash
    # Reinstall plugins
    rm -rf ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    rm -rf ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    # Then repeat the plugin installation steps
    ```

4. Theme display issues:

    - Verify font installation
    - Check terminal emulator settings
    - Ensure terminal supports true color
