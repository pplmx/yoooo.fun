---
title: VLAN
date: 2020-04-18T21:20:37+08:00
slug: 36dff8e50ad540a6afa36151873ad028
draft: false
lastmod: 2020-04-19T18:12:04+08:00
categories: [network]
tags: [device]
keywords: vlan
description: What's VLAN?
---
# VLAN

>   [Get to Know Basic Knwoledge of VLAN(Part 1)](https://www.utepo.net/article/detail/255.html)
>
>   [Get to Know Basic Knwoledge of VLAN(Part 2)](https://www.utepo.net/article/detail/295.html)

## Pros

1.  reduce broadcast storm

    >   The key advantage of VLAN is that it can isolate the conflict domain as well as broadcast domain. If there are hundreds of hosts in a LAN, the network would be completely paralyzed when a broadcast storm happened. Meanwhile, users can divide the broadcast domain through VLAN, limiting the broadcast to each VLAN, and can not be transferred to cross VLAN. Most important, as consideration to improved security, the broadcasts of different VLANS can not communicate without a layer 3 router.

2.  simplify the administration of the network

    >   One of the best things about VLAN is that it simplifies management. By logically grouping users into the same virtual networks, you make it easy to set up and control your policies at a group level. When users physically move workstations, you can keep them on the same network with different equipment. Or if someone changes teams but not workstations, they can easily be given access to whatever new VLANs they need.