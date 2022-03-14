---
title: trunk in switch
date: 2020-04-18T21:18:09+08:00
slug: aa2af77525d5417425517d8ca07d1006
draft: false
lastmod: 2020-04-28T15:44:02+08:00
categories: [network]
tags: [device]
keywords: trunk port, switch
description: What's Trunk port?
---
# Trunk

## Overview

>   https://wenku.baidu.com/view/6694697d657d27284b73f242336c1eb91a373323.html
>

## vlan between switches

![trunk](/assets/trunk.png)

PC1 sends traffic to PC2 after processing its host routing table. These nodes are in the same VLAN but they are connected to different switches. The basic process:

1.  The Ethernet frame leaves PC1 and is received by Switch 1.
2.  The Switch 1 SAT indicates that the destination is on the other end of the trunk line.
3.  Switch 1 uses the trunking protocol to modify the Ethernet frame by adding the VLAN id.
4.  The new frame leaves the trunk port on Switch1 and is received by Switch 2.
5.  Switch2 reads the VLAN id and strips off the trunking protocol.
6.  The original frame is forwarded to the destination (port 4) based on the SAT of Switch 2.

## Openstack Trunk

![openstack trunk port](/assets/20200424170120849.png)



[Reference from here1](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/presentation-media/Neutron-Trunks.pdf)

[Reference from here2](https://www.oreilly.com/library/view/packet-guide-to/9781449311315/ch04.html)
