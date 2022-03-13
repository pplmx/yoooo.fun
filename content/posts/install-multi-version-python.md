---
title: Let's install multiple python versions in linux
date: 2020-04-25T21:55:12+08:00
slug: cc1b9056bc70732925b5e708439b487b
draft: false
lastmod: 2020-04-30T10:13:20+08:00
categories: [python]
tags: [python]
keywords: python, alternatives, multi versions
description: Let's install multiple python versions in linux
---
# install multiple versions of python

[All Python Released Source](https://www.python.org/ftp/python/)

**The following code has been verified in Centos8.**

```bash
#!/usr/bin/env bash

PYTHON_DIR="/opt/python/"
DOWNLOAD_DIR="/home/download/"
PYTHON_2_HOME="${PYTHON_DIR}/py2"
PYTHON_3_HOME="${PYTHON_DIR}/py3"
PYTHON_2_VERSION="2.7.17"
PYTHON_3_VERSION="3.8.2"

# ************ preparation ************
mkdir -p ${DOWNLOAD_DIR}; mkdir -p ${PYTHON_2_HOME}; mkdir -p ${PYTHON_3_HOME}

if ! [[ -f ${DOWNLOAD_DIR}/Python-${PYTHON_3_VERSION}.tgz ]]; then
    wget -P ${DOWNLOAD_DIR} https://www.python.org/ftp/python/${PYTHON_3_VERSION}/Python-${PYTHON_3_VERSION}.tgz || exit
fi

if ! [[ -f ${DOWNLOAD_DIR}/Python-${PYTHON_2_VERSION}.tgz ]]; then
    wget -P ${DOWNLOAD_DIR} https://www.python.org/ftp/python/${PYTHON_2_VERSION}/Python-${PYTHON_2_VERSION}.tgz || exit
fi

tar -zxvf ${DOWNLOAD_DIR}/Python-${PYTHON_3_VERSION}.tgz -C ${DOWNLOAD_DIR}
tar -zxvf ${DOWNLOAD_DIR}/Python-${PYTHON_2_VERSION}.tgz -C ${DOWNLOAD_DIR}


# ************ install dependency packages ************
yum install -y gcc gcc-c++ automake make autoconf libtool diffutils sudo zlib-devel


# ************ install python 2 ************
cd ${DOWNLOAD_DIR}/Python-${PYTHON_2_VERSION} || return
# if need, you can uncomment the following code
# make clean
./configure --prefix=${PYTHON_2_HOME} --enable-optimizations
make
sudo make install


sleep 10s


# ************ install python 3 ************
cd ${DOWNLOAD_DIR}/Python-${PYTHON_3_VERSION} || return
# if need, you can uncomment the following code
# make clean
./configure --prefix=${PYTHON_3_HOME} --enable-optimizations
make
sudo make install


# ************ manage python version ************
# remove old python version management
alternatives --display python | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python >/dev/null 2>&1
# remove old python2 version management
alternatives --display python2 | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python2 >/dev/null 2>&1
# remove old python3 version management
alternatives --display python3 | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python3 >/dev/null 2>&1

# rebuild all python version management
alternatives --install /usr/bin/python python ${PYTHON_2_HOME}/bin/python2 1
alternatives --install /usr/bin/python python ${PYTHON_3_HOME}/bin/python3 9

alternatives --install /usr/bin/python2 python2 ${PYTHON_2_HOME}/bin/python2 9
alternatives --install /usr/bin/python3 python3 ${PYTHON_3_HOME}/bin/python3 9

```