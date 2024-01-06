---
categories:
    - general
date: 2020-06-23T10:02:28Z
description: How to fix rabbit lost user info after a reboot?
keywords: rabbit, user info, reboot
lastmod: 2024-01-06T22:21:46Z
tags:
    - troubleshooting
    - rabbitmq
title: RabbitMQ lost user info
---



# RabbitMQ

## lost user info

> After rebooting, the rabbit user info lost.

### Cause

> Because RabbitMQ stores info by hostname.
>
> Hostname had been changed, so RabbitMQ getting info by the new hostname failed

### Solution

> To add fixed node

```bash
echo 'NODENAME=rabbit@info' | sudo tee -a /etc/rabbitmq/rabbitmq-env.conf

echo '127.0.0.1 info' | sudo tee -a /etc/hosts

ps axu | grep rabbitmq | awk '{print $2}' | sudo xargs kill -9

sudo service rabbitmq-server start
```
