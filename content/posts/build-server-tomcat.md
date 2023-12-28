---
categories:
    - linux
date: 2017-10-13T10:31:14Z
description: install tomcat on linux
keywords: tomcat, linux
lastmod: 2023-08-18T13:13:20Z
tags:
    - tomcat
title: linux服务器初建之tomcat安装
---



# 安装tomcat

## 安装tomcat

```bash
sudo yum install tomcat
```

## 安装管理包

```bash
sudo yum install tomcat-webapps tomcat-admin-webapps
```

<!-- more -->

## 安装在线文档

```bash
sudo yum install tomcat-docs-webapp tomcat-javadoc
```

## 配置tomcat管理页面

```bash
sudo vi /usr/share/tomcat/conf/tomcat-users.xml
```

```text
<tomcat-users>
    <user username="admin" password="admin" roles="manager-gui,admin-gui"/>
</tomcat-users>
```

## 重启

```bash
sudo service tomcat restart
```

## 访问试试

```text
http://yourIP:8080
http://yourIP:8080/manager/html
```
