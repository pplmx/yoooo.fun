---
title: Switch
date: 2020-04-18T21:08:13+08:00
slug: 71d4ff32808ffe99457dcca86a1109e1
draft: false
lastmod: 2020-04-19T18:12:04+08:00
categories: [network]
tags: [device]
keywords: switch, trunk port, access port
description: What's switch?
---
# switch

## Overview

### Link Type

>   The link type of VLAN can be divided into **access link** and **trunk link**.

#### Access Link

-   Access link is part of only one VLAN, and normally is for end devices.
-   Any device attached to an access link is unaware of a VLAN membership.
-   An access-link connection can understand only standard Ethernet frames.
-   Switches remove any VLAN information from the frame before it is sent to an access-link device.

#### Trunk Link

-   Trunk link can carry multiple VLAN traffic and normally is used to connect switches to other switches or to routers.

## Access Port

l Belong to **one** VLAN.

l Commonly used to connect computer ports.

---

-   Strip the VLAN information in the packet and forward the packet directly.

## Trunk Port

l Allow **multiple** VLANs through.

l Receive and send **multiple** VLAN packets.

l Typically used for connection between switches.

---

1.  Compare the PVID of the port and the VLAN information in the packet to be transmitted.

2.  If they are the same, proceed to Step 3, otherwise, proceed to Step 4

3.  Strip the VLAN information in the packet and forward  the packet.

4.  Forward the packet directly.

## Hybrid Port

l Allow **multiple** VLANs through.

l Receive and send **multiple** VLAN packets.

l Used for connection between switches, or switch and computer.

---

1.  Check the VLAN attributes on this port by running the command disp interface to se whether the VLAN attributes is "tagged" or "untagged"

2.  If I is untagged, proceed to Step 3, if it is tagged, proceed to step 4.

3.  Strip the VLAN information in the packet and forward the packet.

4.  Forward the packet directly.

## Summary

| Port Type | Support Mode                                 | Common use cases                                             | Comment                                                      |
| --------- | :------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Access    | single untagged VLAN                         | PC/Printer to switch                                         |                                                              |
| Trunk     | single untagged VLAN & multiple tagged VLANs | switch/hypervisor to switch                                  | VLAN 1 can be Tagged (Untagged by default)                   |
| Hybrid    | Support Untagged VLANs & Tagged VLANs        | 1. Physical Connection: IP Phone to Network Switch Port & a PC to IP Phone’s Switch port; 2. Logical Connection: Voice VLAN as Tagged & Data VLAN as Untagged & Switch port in Trunk mode | 1. Usually the Untagged VLAN number = Native/Default VLAN number; 2. Support for multi-Untagged Frames, usually require the use of protocol-based VLANs; 3. VLAN 1 can be Tagged (Untagged by default) |

![switch-egg](assets/switch-egg.jpg)


[Reference from here]: https://www.utepo.net/article/detail/251.html	"Difference among Switch ports"
