---
title: ansible
date: 2020-04-18T20:24:55+08:00
slug: 640c8a5376aa12fa15cf02130ce239a6
draft: false
lastmod: 2020-04-30T10:13:20+08:00
categories: [automation]
tags: [ansible]
keywords: ansible, directory layout, hello world
description: Let's start a simple sample in Ansible
---
# ansible

>   [Directory Layout](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout)

## Directory Layout

```text
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
   group1.yml             # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```

roles下的目录含义:

`files`：用来存放由copy模块或script模块调用的文件。
`templates`：用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件。
`tasks`：此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件。
`handlers`：此目录应当包含一个main.yml文件，用于定义此角色中触发条件时执行的动作。
`vars`：此目录应当包含一个main.yml文件，用于定义此角色用到的变量。
`defaults`：此目录应当包含一个main.yml文件，用于为当前角色设定默认变量。
`meta`：此目录应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系。

## Alternative Directory Layout

```text
inventories/
   production/
      hosts               # inventory file for production servers
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         hostname1.yml    # here we assign variables to particular systems
         hostname2.yml

   staging/
      hosts               # inventory file for staging environment
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         stagehost1.yml   # here we assign variables to particular systems
         stagehost2.yml

library/
module_utils/
filter_plugins/

site.yml
webservers.yml
dbservers.yml

roles/
    common/
    webtier/
    monitoring/
    fooapp/
   
```

# env

```bash
./
├── filter_plugins
├── group_vars
├── host_vars
│   └── hosts
├── library
├── module_utils
├── roles
└── site.yaml

6 directories, 2 files
```

# ansible cli

```bash
# find system serial
# --module-name or -m    --args or -a
ansible localhost --module-name setup --args 'filter=ansible_product_serial'

# list groups
ansible localhost -m debug -a 'var=groups'

# list groups keys
ansible localhost -m debug -a 'var=groups.keys()'

# list groups(or this command)
ansible-inventory -i inventory/prod.yml --list

```

# hello world

```yaml
---
-
    name: This is a hello-world example
    hosts: localhost
    gather_facts: no
    tasks:
        -
            set_fact:
                hello: 'hello world'
        -
            name: Create a file called '/tmp/testfile.txt' with the content 'hello world'.
            copy:
            	# get variable from hostvars
                content: '{{ hostvars[inventory_hostname]["hello"] }}'
                dest: /tmp/testfile.txt

```

# append some lines to a test file

```yaml
---
-
    name: This is a hello-world example
    hosts: localhost
    vars:
        device_by_pci_address: "{{
            ansible_facts | json_query('@.* | [?pciid].{key: pciid, value: device}') | items2dict
        }}"
    tasks:
        -
            name: To set some variables to hostvars
            set_fact:
                classmates:
                    -
                        sex: 'male'
                        name: 'AA'
                        age: 15
                    -
                        sex: 'female'
                        name: 'BB'
                        age: 16
                    -
                        sex: 'male'
                        name: 'CC'
                    -
                        sex: 'female'
                        name: 'DD'
                    -
                        sex: 'male'
                        name: 'lo'
                    -
                        sex: 'female'
                        name: 'enp0s3'
                pci_bus_addr2nic: "{{ ansible_facts | json_query('@.* | [?pciid].{key: pciid, value: device}') | items2dict }}"

        -
            name: Get system serial
            become: true
            shell: cat /sys/devices/virtual/dmi/id/product_serial
            register: system_serial

        -
            name: To create a test file
            file:
                path: /tmp/testfile.txt
                state: touch
                owner: root
                group: root
                mode: 0777

        -
            name: To append some lines to the test file
            lineinfile:
                # test: get the mac address of a nic
                line: '{{ item.name }}: {{ item.sex }} ==> {{ ansible_facts[item.name].macaddress }} '
                dest: /tmp/testfile.txt
            loop: '{{ hostvars[inventory_hostname]["classmates"] }}'
            when: inventory_hostname not in ['host1', 'host2'] and item.name in ['enp0s3']
        -
            name: To add a block to a file
            blockinfile:
                dest: /tmp/testfile.txt
                block: |
                    hello world
                    Java is the best.
                    system_serial.stdout: {{ system_serial.stdout }}

                    {{ hostvars[inventory_hostname].system_serial.stdout }}
                    #########
                    pci_bus_addr2nic: {{ pci_bus_addr2nic }}

                    {{ hostvars[inventory_hostname]["pci_bus_addr2nic"] }}
                    ###########
                    device_by_pci_address: {{ device_by_pci_address }}

                    Must not Get from this way: hostvars[inventory_hostname]["device_by_pci_address"]
        -
            debug:
                var: pci_bus_addr2nic
        -
            debug:
                var: device_by_pci_address
        -
            debug:
                # this variable is from ansible_facts
                # you can get some info by this command (ansible localhost --module-name setup --args 'filter=ansible_product_serial')
                var: ansible_product_serial
        -
            debug:
                var: system_serial.stdout



```

## install a package on RHEL

```yaml
---
-
    name: install Apache webserver
    hosts: webservers
    tasks:
        -
            name: install httpd
            dnf:
                name: httpd
                State: latest
```

## install a package on Debian

```yaml
---
-
    name: install Apache webserver
    hosts: databases
    tasks:
        -
            name: install Apache webserver
            apt:
                name: apache2
                State: latest
```

## operate a service

```yaml
---
# https://medium.com/bigpanda-engineering/using-ansible-to-compile-nginx-from-sources-with-custom-modules-f6e6c6a42493
-
    name: Start service httpd, if not started
    service:
        name: httpd
        state: started
---
-
    name: Stop service httpd
    service:
        name: httpd
        state: stopped
---
-
    name: Restart network service for interface eth0
    service:
        name: network
        state: restarted
        args: enp2s0
```
