---
categories:
    - network
date: 2023-11-24T10:45:42Z
description: Network Availability on Linux and Windows.
keywords:
    - network
    - linux
    - windows
    - nc
    - nmap
    - telnet
    - Test-NetConnection
    - powershell
    - pwsh
lastmod: 2023-11-24T10:45:42Z
tags:
    - network
    - linux
    - windows
title: Check Network Availability
---



# Check Network Availability

The following commands can be used to check if a remote host is available on the network.

Here is an example to check if port 22 on 192.168.21.32 is open.

## Linux

```shell
nc -zv 192.168.21.32 22

# or
nmap -p 22 192.168.21.32

# or, if no output, port is open
timeout 1 bash -c 'cat < /dev/null > /dev/tcp/192.168.21.32/22'
echo >/dev/tcp/192.168.21.32/22
```

## Windows

```shell
telnet 192.168.21.32 22

# or (PowerShell)
Test-NetConnection -ComputerName 192.168.21.32 -Port 22
```
