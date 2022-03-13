---
title: Run a specified role in ansible
date: 2020-04-30T09:59:05+08:00
lastmod: 2020-05-11T13:55:25+08:00
slug: 04b1bf529f7636034eb1e8ac03a4f75c
draft: false
categories: [automation]
tags: [ansible]
keywords: ansible, role, tags
description: Let's run a specified role in Ansible
---
# install lldpd by ansible roles

## ansible layout

```bash
[root@03e6e45a7f92 cc]# tree ansible
ansible
├── roles
│   └── lldp
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       └── tasks
│           ├── disable_lldp.yml
│           ├── enable_lldp.yml
│           └── main.yml
└── site.yml

5 directories, 5 files
```

## show the content

### site.yml

```yaml
---
-
    hosts: localhost
    name: lldpd installation
    roles:
        -
            role: lldp
            tags: lldp

```
---
Or like this
```yaml
---
-
    hosts: localhost
    name: lldpd installation
    tasks:
        -
            import_role:
                name: lldp
            tags: lldp
```

### defaults/main.yml

```yaml
---
LLDPD_VERSION: 1.0.5
```

### tasks/main.yml

```yaml
---
-
    include: enable_lldp.yml
```

### tasks/enable_lldp.yml

```yaml
---
-
    name: Retrieve lldpd source code
    get_url:
        # TODO replace it
        url: https://media.luffy.cx/files/lldpd/lldpd-{{ LLDPD_VERSION }}.tar.gz
        dest: /tmp/lldpd-{{ LLDPD_VERSION }}.tar.gz
-
    name: Extract archive
    unarchive:
        # if configured like as the following, 'Retrieve lldpd source code' task can be removed
        # src: https://media.luffy.cx/files/lldpd/lldpd-{{ LLDPD_VERSION }}.tar.gz
        src: /tmp/lldpd-{{ LLDPD_VERSION }}.tar.gz
        dest: /tmp
-
    name: Configure install
    command: ./configure
    args:
        chdir: /tmp/lldpd-{{ LLDPD_VERSION }}
        creates: /tmp/lldpd-{{ LLDPD_VERSION }}/configure.status.log
-
    name: Build lldpd
    command: make
    args:
        chdir: /tmp/lldpd-{{ LLDPD_VERSION }}
        creates: /tmp/lldpd-{{ LLDPD_VERSION }}/make.status.log
-
    name: Install lldpd
    become: true
    command: make install
    args:
        chdir: /tmp/lldpd-{{ LLDPD_VERSION }}
        creates: /tmp/lldpd-{{ LLDPD_VERSION }}/make_install.status.log
-
    name: Create chroot for lldpd
    file:
        path: /usr/local/var/run/lldpd
        state: directory
-
    name: Create _lldpd group
    group:
        name: _lldpd
        state: present
-
    name: Create _lldpd user
    user:
        name: _lldpd
        group: _lldpd
        comment: lldpd user
-
    name: Remove build directory
    file:
        path: /tmp/lldpd-{{ LLDPD_VERSION }}
        state: absent
-
    name: Remove archive
    file:
        path: /tmp/lldpd-{{ LLDPD_VERSION }}.tar.gz
        state: absent
-
    name: Create lldpd.conf
    file:
        path: /etc/lldpd.conf
        state: touch
-
    name: Configure lldpd.conf
    blockinfile:
        path: /etc/lldpd.conf
        block: |
            # Reference from https://vincentbernat.github.io/lldpd/usage.html
            configure system chassisid <chassisid>
            configure system hostname <hostname>
            configure system description <description>
            configure system platform <platform>

            configure system interface pattern eth0,eth1
            configure system interface permanent eth0,eth1

            # rx-and-tx: receive and transmit LLDP frames
            # tx-only:
            # rx-only:
            # disabled:
            configure ports eth0,eth1 lldp status tx-only

            # TTL is tx-interval * tx-hold, i.e. 120 seconds
            configure lldp tx-interval <number, default is 30 seconds>
            configure lldp tx-hold <number, default is 4>
-
    name: Enable lldpd service
    systemd:
        name: lldpd
        enabled: true
        masked: false
-
    name: Start lldpd service
    systemd:
        name: lldpd
        state: started


```

## run

```bash
ansible-playbook site.yml
# or only run role: lldp
ansible-playbook site.yml -t lldp
```