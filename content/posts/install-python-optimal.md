---
title: Install Python from Source Code
date: 2020-09-27T13:32:10+08:00
lastmod: 2020-10-28T14:02:27+08:00
slug: 7dd3e8847edb1f9059597408705095bf
draft: false
categories: [python]
tags: [python]
keywords: python, alternatives, multi versions
description: Automatically install Python, just need to set python version number
---
# install Python(Optimization Version)

```bash
#!/usr/bin/env bash

PYTHON_DIR="/opt/python"
DOWNLOAD_PYTHON_DIR="/home/download/python"

# create the dirs
install -d ${DOWNLOAD_PYTHON_DIR}

PYTHON_DEFAULT_VERSION="3.8.6"

function install_python() {
    # please ensure the version you specified lists here
    # https://www.python.org/ftp/python/
    python_version=${1:-$PYTHON_DEFAULT_VERSION}
    python_home="${PYTHON_DIR}/${python_version}"

    # create the dirs
    install -d "${python_home}"

    python_remote_url="https://www.python.org/ftp/python/${python_version}/Python-${python_version}.tgz"
    python_local_url="${DOWNLOAD_PYTHON_DIR}/Python-${python_version}.tgz"

    [[ ! -f ${python_local_url} ]] && wget -P ${DOWNLOAD_PYTHON_DIR} "${python_remote_url}"

    tar -zxvf "${python_local_url}" -C ${DOWNLOAD_PYTHON_DIR} || exit

    # ************ install dependency packages ************
    yum install -y gcc gcc-c++ automake make autoconf libtool diffutils sudo zlib-devel || exit

    # ************ install python ************
    cd "${DOWNLOAD_PYTHON_DIR}/Python-${python_version}" || return
    # if need, you can uncomment the following code
    # make clean
    ./configure --prefix="${python_home}" --enable-optimizations
    make
    sudo make install

    # export to path
    PY_BIN="/opt/python/${python_version}/bin"
    if [[ ${SHELL} =~ "/bin/zsh" ]]; then
        [[ ! ${PATH} =~ ${PY_BIN} ]] && echo "PATH=/opt/python/${python_version}/bin/:\$PATH" >>"${HOME}/.zshrc"
        # shellcheck source=$HOME
        source "${HOME}/.zshrc"
        export PATH
    elif [[ ${SHELL} =~ "/bin/bash" ]]; then
        [[ ! ${PATH} =~ ${PY_BIN} ]] && echo "PATH=/opt/python/${python_version}/bin/:\$PATH" >>"${HOME}/.bashrc"
        # shellcheck source=$HOME
        source "${HOME}/.bash_profile"
    else
        return
    fi

    manage_python
}

function manage_python() {
    # remove old python version management
    alternatives --display python | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python >/dev/null 2>&1
    alternatives --display pip | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove pip >/dev/null 2>&1

    # rebuild new python version
    py_v="python${python_version}"
    pip_v="pip${python_version}"
    if [[ ${python_version} == 2* ]]; then
        alternatives --display python2 | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python2 >/dev/null 2>&1

        rm -fr /usr/bin/python
        rm -fr /usr/bin/pip
        rm -fr /usr/bin/python2
        rm -fr /usr/bin/pip2

        # manage python
        alternatives --install "/usr/bin/${py_v}" "${py_v}" "${python_home}/bin/python2" 9
        alternatives --install /usr/bin/python2 python2 "/usr/bin/${py_v}" 9
        alternatives --install /usr/bin/python python /usr/bin/python2 1
    fi

    if [[ ${python_version} == 3* ]]; then
        alternatives --display python3 | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove python3 >/dev/null 2>&1
        alternatives --display pip3 | grep priority | awk '{print $1}' | xargs -n1 alternatives --remove pip3 >/dev/null 2>&1

        rm -fr /usr/bin/python
        rm -fr /usr/bin/pip
        rm -fr /usr/bin/python3
        rm -fr /usr/bin/pip3

        # manage python
        alternatives --install "/usr/bin/${py_v}" "${py_v}" "${python_home}/bin/python3" 9
        alternatives --install /usr/bin/python3 python3 "/usr/bin/${py_v}" 9
        alternatives --install /usr/bin/python python /usr/bin/python3 9

        # manage pip
        alternatives --install "/usr/bin/${pip_v}" "${pip_v}" "${python_home}/bin/pip3" 9
        alternatives --install /usr/bin/pip3 pip3 "/usr/bin/${pip_v}" 9
        alternatives --install /usr/bin/pip pip /usr/bin/pip3 9
    fi
}

# Usage:
#     default install python 3.8.6
#     sh install_python.sh
#     sh install_python.sh 3.9.0
install_python "$@"

```