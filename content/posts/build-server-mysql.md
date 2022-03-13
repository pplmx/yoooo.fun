---
title: linux服务器初建之mysql安装
date: 2017-10-13T08:33:23+08:00
slug: 467986a3333fded16813a0238a76f154
draft: false
lastmod: 2020-01-16T21:25:04+08:00
categories: [linux]
tags: [database]
keywords: mysql, linux
description: install mysql on linux
---
# 安装mysql
## 下载mysql的repo源
```bash
$ wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```
## 安装mysql的rpm包
```bash
$ sudo rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```
## 安装mysql
```bash
$ sudo yum install mysql-server
```
<!-- more -->
## 重置mysql密码
```bash
$ mysql -u root
```
登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ 
原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
```bash
$ sudo chown -R root:root /var/lib/mysql
```
重启mysql服务
```bash
$ service mysqld restart
```
接下来是登录mysql(有两种情况)
### 直接登录成功
```bash
$ mysql -u root  //直接回车进入mysql控制台
mysql > use mysql;
mysql > update user set password=password('123456') where user='root';
mysql > exit;
```
### 登录失败
登录失败:是因为密码错误,不是默认的空密码,
而是在安装时,mysql默认分配了随机密码
如果你的安装信息是详细显示的,那么你是可以在之前的安装信息中,找到随机密码
找不到,那就继续如下操作:

1.修改MySQL的登录设置：
```bash
# vi /etc/my.cnf // 在[mysqld]的段中加上一句：skip-grant-tables
```
2.重新启动mysql,并登录(mysql5.7,password字段不存在了,而是authentication_string)
```bash
# service mysqld restart
# mysql -uroot -p//回车
mysql> use mysql;
mysql> update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';
mysql> flush privileges;
mysql> quit; 
```
3.还原/etc/my.cnf(将skip-grant-tables删除)

4.重启mysql,即可使用新密码登录了
