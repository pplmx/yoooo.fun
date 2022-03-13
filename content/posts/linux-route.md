---
title: set route on linux
date: 2019-05-31T20:13:55+08:00
lastmod: 2020-03-25T20:23:46+08:00
slug: 3359169a677bbc26a7757faf5b035b86
draft: false
categories: [linux]
tags: [network]
keywords: route, network, linux
description: some linux route operations
---

# linux route

## add static route

- add default static route

```bash
# Permanent
echo "any net 0.0.0.0/0 gw 110.188.40.1" >> /etc/sysconfig/static-routes
# Temporary
ip route add default dev vlan7
ip route add default via 110.188.40.1
ip route add default via 110.188.40.1 dev vlan7
ip route add 0.0.0.0/0 dev vlan7
ip route add 0.0.0.0/0 via 110.188.40.1
ip route add 0.0.0.0/0 via 110.188.40.1 dev vlan7
```

<!-- more -->

- add specific net static route

```bash
echo "any net 110.188.40.0/24 gw 110.188.40.1" >> /etc/sysconfig/static-routes
```

## delete route

```bash
# delete default route
route del default gw 110.188.40.1
ip route del default via 110.188.18.1 dev vlan16
# delete a non-default route
ip route del 110.188.18.0/24 via 110.188.18.1 dev vlan16
```

## replace route

```bash
# work well after every network restart
# replace if exists, or add
echo "ip route replace default via 110.188.40.1 dev vlan7" >> /sbin/ifup-local
chmod +x /sbin/ifup-local
systemctl restart network
```

## change route

```bash
# change some params of existing route
ip route change 192.192.13.1/24 dev ens32
```
