---
title: Let's understand what's lldp.
date: 2020-04-18T21:08:20+08:00
slug: afd61c5e54eeb52aeafbfb646bbcd0af
draft: false
lastmod: 2020-04-19T18:12:04+08:00
categories: [network]
tags: [protocol]
keywords: lldp, TCP/IP
description: What's LLDP?
---
# LLDP

## Overview

目前，网络设备的种类日益繁多且各自的配置错综复杂，为了使不同厂商的设备能够在网络中相互发现并交互各自的系统及配置信息，需要有一个标准的信息交流平台。

LLDP（Link Layer Discovery Protocol，**链路层发现协议**）就是在这样的背景下产生的，它提供了一种标准的链路层发现方式，可以将本端设备的**主要能力**、**管理地址**、**设备标识**、**接口标识**等信息组织成不同的`TLV`（Type/Length/Value，类型/长度/值），并封装在LLDPDU（LLDP Data Unit，**链路层发现协议数据单元**）中发布给与自己直连的邻居，邻居收到这些信息后将其以标准MIB（Management Information Base，**管理信息库**）的形式保存起来，以供网络管理系统查询及判断链路的通信状况。

类似的还有Cisco Discovery Protocol, Foundry Discovery Protocol, Nortel Discovery Protocol, etc.

## Frame Structure

LLDP的以太帧通常有DST MAC, SRC MAC, EtherType(0x88cc). 以及LLDPDU和FCS组成.

**Ethernet Frame structure**:

| [Preamble](https://en.wikipedia.org/wiki/Preamble_(communication)) |                       Destination MAC                        |    Source MAC     | [Ethertype](https://en.wikipedia.org/wiki/Ethertype) | LLDPDU | [Frame check sequence](https://en.wikipedia.org/wiki/Frame_check_sequence) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :---------------: | :--------------------------------------------------: | ------ | :----------------------------------------------------------: |
|                                                              | 01:80:c2:00:00:0e, or 01:80:c2:00:00:03, or 01:80:c2:00:00:00 | Station's address |                        0x88CC                        |        |                                                              |

**LLDPDU**:

| Chassis ID TLV | Port ID TLV | Time to live TLV | Optional TLV(s) | End of  LLDPDU TLV |
| -------------- | ----------- | ---------------- | --------------- | ------------------ |
| Type=1         | Type=2      | Type=3           | Optional TLV    | ...                |

**TLV structures**:

|  Type  | Length |    Value     |
| :----: | :----: | :----------: |
| 7 bits | 9 bits | 0-511 octets |

**TLV type values**:

| TLV type |      TLV name       | Usage in LLDPDU |
| :------: | :-----------------: | :-------------: |
|    0     |    End of LLDPDU    |    Mandatory    |
|    1     |     Chassis ID      |    Mandatory    |
|    2     |       Port ID       |    Mandatory    |
|    3     |    Time To Live     |    Mandatory    |
|    4     |  Port description   |    Optional     |
|    5     |     System name     |    Optional     |
|    6     | System description  |    Optional     |
|    7     | System capabilities |    Optional     |
|    8     | Management address  |    Optional     |
|  9–126   |      Reserved       |        -        |
|   127    |     Custom TLVs     |    Optional     |

Custom TLVs are supported via a TLV type 127. The value of a custom TLV starts with a 24-bit organizationally unique identifier and a 1 byte organizationally specific subtype followed by data. The basic format for an organizationally specific TLV is shown below:

|    Type    | Length | Organizationally unique identifier (OUI) | Organizationally defined subtype | Organizationally defined information string |
| :--------: | :----: | :--------------------------------------: | :------------------------------: | :-----------------------------------------: |
| 7 bits—127 | 9 bits |                 24 bits                  |              8 bits              |                0-507 octets                 |

According to IEEE Std 802.1AB, §9.6.1.3, "The Organizationally Unique Identifier shall contain the organization's OUI as defined in IEEE Std 802-2001." Each organization is responsible for managing their subtypes.

## Work Mechanism

### LLDP工作模式

-   TxRx: 既发送也接收LLDP报文
-   Tx: 只发送LLDP报文
-   Rx: 只接收LLDP报文
-   Disable: 既不发送也不接收LLDP报文

当端口的LLDP工作模式发生变化时，端口将对协议状态机进行初始化操作。为了避免端口工作模式频繁改变而导致端口不断执行初始化操作，可配置端口初始化延迟时间，当端口工作模式改变时延迟一段时间再执行初始化操作。

### LLDP发送机制

>   当端口工作在TxRx或Tx模式时，设备会周期性地向邻居设备发送LLDP报文。如果设备的本地配置发生变化则立即发送LLDP报文，以将本地信息的变化情况尽快通知给邻居设备。但为了防止本地信息的频繁变化而引起LLDP报文的大量发送，每发送一个LLDP报文后都需延迟一段时间后再继续发送下一个报文。
>
>   当设备的工作模式由Disable/Rx切换为TxRx/Tx，或者发现了新的邻居设备（即收到一个新的LLDP报文且本地尚未保存发送该报文设备的信息）时，该设备将自动启用快速发送机制，即将LLDP报文的发送周期缩短为1秒，并连续发送指定数量的LLDP报文后再恢复为正常的发送周期。

### LLDP接收机制

>   当端口工作在TxRx或Rx模式时，设备会对收到的LLDP报文及其携带的TLV进行有效性检查，通过检查后再将邻居信息保存到本地，并根据TTL（Time To Live，生存时间） TLV中TTL的值来设置邻居信息在本地设备上的老化时间，若该值为零，则立刻老化该邻居信息。



[H3C LLDP Information](http://www.h3c.com/cn/d_200805/605853_30003_0.htm)

[LLDP wiki](https://en.wikipedia.org/wiki/Link_Layer_Discovery_Protocol)