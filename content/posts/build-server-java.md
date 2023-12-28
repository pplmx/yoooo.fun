---
categories:
    - linux
date: 2017-10-13T09:48:25Z
description: install java on linux
keywords: java, linux
lastmod: 2023-08-18T13:13:20Z
tags:
    - java
title: linux服务器初建之java环境
---



# 安装JDK

如果命令显示无权限,请加sudo,或是进入root用户;

若是目录无权限访问,请给相关用户设置相应的访问权限-rwx

## 查看是否存在jdk环境

(根据需求决定是否卸载已存在的环境)

```bash
yum list installed |grep java
yum -y remove java-1.8.0-openjdk*  # 匹配所有以java-1.8.0-openjdk开头的文件,然后卸载
```

<!-- more -->

## 安装jdk

```bash
yum -y list java*  # 查看jdk软件包列表
yum install java-1.8.0-openjdk*  # 安装所有java程序
# 如果不是所有程序都需要,也可仅执行如下命令
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
java -version  # 查看jdk版本号
```

## 配置环境变量

```bash
vi /etc/profile # 编辑该文件
```

```text
# set java environment
JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.144-0.b01.el7_4.x86_64
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME  CLASSPATH  PATH
```

退出并保存

```bash
. /etc/profile   # 注意:那里是需要空格的
echo $JAVA_HOME  # 查看
echo $CLASSPATH  # 查看
```
