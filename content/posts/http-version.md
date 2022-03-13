---
title: Http Version Comparison
date: 2020-09-16T10:53:10+08:00
slug: 215a8e96a09ea1b4b76a8b7fa6ddc142
draft: false
lastmod: 2020-09-16T11:02:02+08:00
categories: [network]
tags: [http]
keywords: http/1.0, http/1.1, http/2.0, http/3.0
description: HTTP Evolution in 1.0, 2.0, 3.0
---

# HTTP

## Preview

| Year |                         HTTP Version                         |
| :--: | :----------------------------------------------------------: |
| 1996 |          [1.0](https://tools.ietf.org/html/rfc1945)          |
| 1997 |          [1.1](https://tools.ietf.org/html/rfc2616)          |
| 2000 |         [HTTPS](https://tools.ietf.org/html/rfc2818)         |
| 2015 |          [2.0](https://tools.ietf.org/html/rfc7540)          |
| ???  | [3.0 Draft](https://tools.ietf.org/html/draft-ietf-quic-http) |

![HTTP_Version](assets/HTTP-v1-v2-v3-stacks.png)

|                                             |                                                 |                                             |
| ------------------------------------------- | ----------------------------------------------- | ------------------------------------------- |
| ![HTTP1 Protocol](assets/http1-265x300.png) | ![HTTP1.1 Protocol](assets/http1.1-265x300.png) | ![HTTP2 Protocol](assets/http2-283x300.png) |

- HTTP/1.0

    > For every TCP connection there is only one request and one response.

- HTTP/1.1

    > It supports connection reuse i.e. for every TCP connection there could be multiple requests and responses, and pipelining where the client can request several resources from the server at once.
    >
    > However, pipelining was hard to implement due to issues such as head-of-line blocking and was not a feasible solution.

- HTTP/2

    > Uses multiplexing, where over a single TCP connection resources to be delivered are interleaved and arrive at the client almost at the same time.
    >
    > It is done using streams which can be prioritized, can have dependencies and individual flow control.
    >
    > It also provides a feature called server push that allows the server to send data that the client will need but has not yet requested.

![Comparison of HTTP versions](assets/Comparison-of-HTTP-versions.jpg)

## http 1.0

- No Connection

    Each request from the browser need build a connection with the server, once the server has handled the request and it will stop the tcp connection immediately.

- No State

    The server do not trace every client, and record the past requests too.

## http 1.1

- persistent connection
- Host header is required
- pipelining
- cache-control
- content negotiation

## https

https ==> HTTP + SSL

## http 2.0

- Binary Protocol
  - Low overhead in parsing data — **a critical value proposition in HTTP/2 vs HTTP1**.
  - Less prone to errors.
  - Lighter network footprint.
  - Effective network resource utilization.
  - Reduced network latency and improved throughput.
  - Eliminating security concerns associated with the textual nature of HTTP1.x such as response  splitting attacks.
  - Efficient and robust in terms of processing of data between client and server.
  - Compact representation of commands for easier processing and implementation.
  - Enables other capabilities of the HTTP/2 including compression, multiplexing, prioritization, flow control and effective handling of TLS.
- Request Multiplexing
  - Allows you to download web files asynchronously from one server.
- Header Compression
- Server Push
  - The client saves pushed resources in the cache.
  - The client can reuse these cached resources across different pages.
  - The server can multiplex pushed resources along with originally requested information within the  same TCP connection.
  - The server can prioritize pushed resources — **a key performance differentiator in HTTP/2 vs HTTP1**.
  - The client can decline pushed resources to maintain an effective repository of cached resources or  disable Server Push entirely.
  - The client can also limit the number of pushed streams multiplexed concurrently.

## http 3.0