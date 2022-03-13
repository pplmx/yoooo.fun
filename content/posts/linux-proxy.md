---
title: proxy in linux
date: 2020-04-18T21:00:15+08:00
lastmod: 2020-04-19T18:12:04+08:00
slug: 2785cfce9493dbf7745d9c15b2edcbaf
draft: false
categories: [linux]
tags: [proxy]
keywords: linux, proxy
description: set proxy for linux
---
# proxy for linux(centos/rhel)

Define the environment variables in /etc/environment file if you want to add a permanent proxy in the CentOS/RHEL 7.

```bash
echo "all_proxy=http://proxy.example.com:3128/" > /etc/environment
```



For **bash** and **sh** users, add the export line given above into a new file called **/etc/profile.d/http_proxy.sh** file:

```bash
echo "export all_proxy=http://proxy.example.com:3128/" > /etc/profile.d/http_proxy.sh
```

## `PS`

| Env Variable | Desc                                                         | e.g.                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| http_proxy   |                                                              | 10.0.0.51:8080<br/>http://10.0.0.51:8080<br/>user:pass@10.0.0.10:8080<br/>socks4://10.0.0.51:1080<br/>socks5://192.168.1.1:1080 |
| https_proxy  |                                                              | ditto                                                        |
| ftp_proxy    |                                                              | ditto                                                        |
| all_proxy    | if this variable is set, there is no need to set the above variables | ditto                                                        |
| no_proxy     |                                                              | *.aiezu.com,10.*.*.*,192.168.*.*,*.local,localhost,127.0.0.1 |
