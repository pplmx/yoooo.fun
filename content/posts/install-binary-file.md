---
categories:
    - linux
date: 2023-03-10T16:45:12Z
description: install golang and node binary packages on Unix-like systems
keywords: binary,linux,go,node
lastmod: 2023-08-18T10:45:12Z
tags:
    - binary
    - linux
    - go
    - node
title: Install binary packages on Unix-like systems
---



```shell
export TMP_DIR="/var/tmp"
export DOWNLOAD_URL_GO="https://go.dev/dl/go1.21.0.linux-amd64.tar.gz"
export DOWNLOAD_URL_NODE="https://nodejs.org/dist/v20.5.1/node-v20.5.1-linux-x64.tar.xz"
export TAR_GO="go.tar.gz"
export TAR_NODE="node.tar.xz"

# by wget
wget -O ${TMP_DIR}/${TAR_GO} ${DOWNLOAD_URL_GO}
wget -O ${TMP_DIR}/${TAR_NODE} ${DOWNLOAD_URL_NODE}

# by curl
curl -fsSL -o ${TMP_DIR}/${TAR_GO} ${DOWNLOAD_URL_GO}
curl -fsSL -o ${TMP_DIR}/${TAR_NODE} ${DOWNLOAD_URL_NODE}

# by aria2
rm -fr ${TMP_DIR}/${TAR_GO} && aria2c -d ${TMP_DIR} -o ${TAR_GO} ${DOWNLOAD_URL_GO}
rm -fr ${TMP_DIR}/${TAR_NODE} && aria2c -d ${TMP_DIR} -o ${TAR_NODE} ${DOWNLOAD_URL_NODE}

# If not set to -z, or -j, or -J, etc., it will automatically decompress the files by file extension.
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xf ${TMP_DIR}/${TAR_GO}
sudo rm -fr /usr/local/node && sudo install -d /usr/local/node && sudo tar -C /usr/local/node -xf ${TMP_DIR}/${TAR_NODE} --strip-components=1

# You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation):
# for me, I like add exports to /etc/profile.d/sh.local for a system-wide
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:/usr/local/node/bin

# Verify it
go version
node -v
```
