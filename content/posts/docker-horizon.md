---
title: Horizon Image Based on Mitaka
date: 2020-08-27T17:06:35+08:00
slug: 5bf999f26e032a3831707c8be992934f
draft: false
lastmod: 2020-08-27T17:14:39+08:00
categories: [openstack]
tags: [horizon,dashboard]
keywords: OpenStack, horizon, dashboard
description: A simple horizon docker demo. FYI.
---

# Horizon

**All accounts' password is `root`.**

```bash
# create container and expose some ports
docker container run -d --privileged --name ho \
    -p 80:80 -p 5000:5000 -p 35357:35357 \
    --add-host info:127.0.0.1 --add-host controller:127.0.0.1 \
    purplemystic/mitaka_horizon init

# restart rabbitmq and apache2
# mysql and apache2 use domain: controller
# rabbitmq use domain: info(For more information: https://blog.caoyu.info/rabbitmq-lost-user-info.html)
docker container exec -it ho bash -c "service mysql restart; service rabbitmq-server restart; service memcached restart; service apache2 restart"

docker container exec -it ho bash
# login to ho, and source openrc
source ~/admin-openrc

# access by browser
# configure your /etc/hosts
echo "127.0.0.1 info controller" >> /etc/hosts
# Finally, you can login by browser
http://127.0.0.1/horizon

```