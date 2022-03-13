---
title: tcpdump
date: 2020-04-18T21:05:32+08:00
lastmod: 2020-04-19T18:12:04+08:00
slug: fcd381f0102bb0271a5b0237cd756ba3
draft: false
categories: [linux]
tags: [cli]
keywords: tcpdump
description: some common tcpdump usages
---
# some common tcpdump cli

```bash
# Reading / Writing Captures to a File 
tcpdump port 80 -w capture_file 

# read PCAP files by using the -r switch 
tcpdump -r capture_file 

# port 2000 of any nic  
tcpdump -i any port 2000 –nn 

# Everything on an interface 
tcpdump -i eth0 

# Find Traffic by IP 
# One of the most common queries, using host, you can see traffic that’s going to or from 1.1.1.1. 
tcpdump host 1.1.1.1 

# Filtering by Source and/or Destination 
tcpdump src 1.1.1.1  
tcpdump dst 1.0.0.1 

# Finding Packets by Network 
tcpdump net 1.2.3.0/24 

# Get Packet Contents with Hex Output 
tcpdump -c 1 -X icmp 

# Show Traffic Related to a Specific Port 
tcpdump port 3389  
tcpdump src port 1025 

# Show Traffic of One Protocol 
tcpdump icmp 

# Show only IP6 Traffic 
tcpdump ip6 

# Find Traffic Using Port Ranges 
tcpdump portrange 21-23 

# Find Traffic Based on Packet Size 
tcpdump less 32  
tcpdump greater 64  
tcpdump <= 128 

# ==================================

# It’s All About the Combinations 

# ========= AND =========
# and or && 

# ========= OR ========= 
# or or || 

# ========= EXCEPT =========
# not or ! 

# From specific IP and destined for a specific Port 
# Let’s find all traffic from 10.5.2.3 going to any host on port 3389. 
tcpdump -nnvvS src 10.5.2.3 and dst port 3389 

# From One Network to Another 
# Let’s look for all traffic coming from 192.168.x.x and going to the 10.x or 172.16.x.x networks
# and we’re showing hex output with no hostname resolution and one level of extra verbosity. 
tcpdump -nvX src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16 

# Non ICMP Traffic Going to a Specific IP 
# This will show us all traffic going to 192.168.0.2 that is not ICMP. 
tcpdump dst 192.168.0.2 && src ! icmp 
tcpdump dst 192.168.0.2 and src not icmp

# catch packages from(to) eth0 or eth1
tcpdump -vi eth0 || eth1 -w tmp.pcap
```

# Advanced

## match MAC address & VLAN

-   ether host <MAC> - capture packets sent from and to <MAC>
-   ether src <MAC> - capture packets sent from <MAC>
-   ether dst <MAC> - capture packets sent to <MAC>
-   vlan <VLAN ID> - match <VLAN ID>

## match protocol

Match protocols in L3 header:

-   `ip proto ` - PROTO: icmp, icmp6, igmp, igrp, pim, ah, esp, vrrp, udp, or tcp

Follow are abbreviations:

-   `icmp` = `proto icmp`
-   `tcp` = `proto tcp`
-   `udp` = `proto udp`

Match protocols in L2 header:

-   `ether proto ` - PROTO: ip, ip6, arp, rarp, atalk, aarp, decnet, sca, lat, mopdl, moprc, iso, stp, ipx, or netbeui

Follow are abbreviations:

-   `ip` = `ether proto ip`
-   `ip6` = `ether proto ip6`
-   `arp` = `ether proto arp`
-   `rarp` = `ether proto rarp`

```bash
tcpdump -i eth0 arp
tcpdump -i eth0 icmp
```
