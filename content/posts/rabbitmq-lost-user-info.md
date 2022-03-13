---
title: RabbitMQ lost user info
date: 2020-06-23T10:02:28+08:00
slug: 9e91456c87b8da5a298415872fdd55ab
draft: false
lastmod: 2020-06-23T10:45:43+08:00
categories: [troubleshooting]
tags: [troubleshooting, rabbitmq]
keywords: rabbit, user info, reboot
description: How to fix rabbit lost user info after a reboot?
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