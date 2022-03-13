---
title: Hexo with Nginx on CentOS
date: 2019-02-12T19:47:51+08:00
slug: 496a857ab31e94bdf445f289c157f037
draft: false
lastmod: 2020-01-16T21:25:04+08:00
categories: [linux]
tags: [hexo,nginx]
keywords: nginx, centos, hexo
description: deploy hexo with nginx on centos
---
# deploy hexo by nginx on centos
- create some dirs and enter pkgs dir
```shell
mkdir /home/packages && cd /home/packages
```
- download npm compiled source code and unzip it
```shell
wget https://nodejs.org/dist/v11.9.0/node-v11.9.0-linux-x64.tar.xz
tar -xvf node-v11.9.0-linux-x64.tar.xz -C /usr/share/
```
<!-- more -->
- create soft link
```shell
mv /usr/share/node-v11.9.0-linux-x64 /usr/share/nodejs
ln -s /usr/share/nodejs/bin/node /usr/local/bin/
ln -s /usr/share/nodejs/bin/npm /usr/local/bin/
```
- install hexo
```shell
npm install -g hexo-cli
ln -s /usr/share/nodejs/bin/hexo /usr/local/bin/
```
---
## install nginx
- configure yum repo
```shell
vi /etc/yum.repos.d/my.repo
```
```xml
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```
- install nginx and start it
```shell
yum install -y nginx
systemctl start nginx && systemctl enable nginx
```
---
- init hexo project
```shell
mkdir /home/website && cd /home/website && hexo init && hexo g
```
- refer to hexo public dir
```shell
vi /etc/nginx/conf.d/default.conf
```
```text
location / {                    
    root   /home/website/public;
    index  index.html index.htm;
}                               
```
```shell
systemctl restart nginx
```
---
---
Now, you can access your blog by your IP address.