---
categories:
    - automation
date: 2020-04-30T09:55:47Z
description: Let's install a software from source in Ansible
keywords: ansible, install from source, lldpd
lastmod: 2020-04-30T10:45:50Z
tags:
    - ansible
title: ansible install software from source
---



# install lldpd from source by ansible

## show the content

```bash
cat lldpd.yml
```

```yml
---
-   hosts: localhost
    name: lldpd installation
    vars:
        lldpd_version: 1.0.5
    tasks:
        -   name: Retrieve lldpd source code
            get_url:
                # TODO replace it
                url: https://media.luffy.cx/files/lldpd/lldpd-{{ lldpd_version }}.tar.gz
                dest: /tmp/lldpd-{{ lldpd_version }}.tar.gz
        -   name: Extract archive
            unarchive:
                # if configured like as the following, 'Retrieve lldpd source code' task can be removed
                # src: https://media.luffy.cx/files/lldpd/lldpd-{{ lldpd_version }}.tar.gz
                src: /tmp/lldpd-{{ lldpd_version }}.tar.gz
                dest: /tmp
        -   name: Configure install
            command: ./configure
            args:
                chdir: /tmp/lldpd-{{ lldpd_version }}
                creates: /tmp/lldpd-{{ lldpd_version }}/configure.status.log
        -   name: Build lldpd
            command: make
            args:
                chdir: /tmp/lldpd-{{ lldpd_version }}
                creates: /tmp/lldpd-{{ lldpd_version }}/make.status.log
        -   name: Install lldpd
            become: true
            command: make install
            args:
                chdir: /tmp/lldpd-{{ lldpd_version }}
                creates: /tmp/lldpd-{{ lldpd_version }}/make_install.status.log
        -   name: Create chroot for lldpd
            file:
                path: /usr/local/var/run/lldpd
                state: directory
        -   name: Create _lldpd group
            group:
                name: _lldpd
                state: present
        -   name: Create _lldpd user
            user:
                name: _lldpd
                group: _lldpd
                comment: lldpd user
        -   name: Remove build directory
            file:
                path: /tmp/lldpd-{{ lldpd_version }}
                state: absent
        -   name: Remove archive
            file:
                path: /tmp/lldpd-{{ lldpd_version }}.tar.gz
                state: absent
        -   name: Enable lldpd service
            systemd:
                name: lldpd
                enabled: true
                masked: false
        -   name: Start lldpd service
            systemd:
                name: lldpd
                state: started

```

## run

```bash
ansible-playbook lldpd.yml
```
