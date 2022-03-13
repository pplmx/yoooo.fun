---
title: IDEA对@Autowired的使用提示警告
date: 2017-10-16T15:41:38+08:00
lastmod: 2020-01-16T21:25:04+08:00
slug: c128e70b9e7a30ebfb683056f24bceda
draft: false
categories: [java]
tags: [spring]
keywords: autowired, spring, java
description: warning about @Autowired
---
# idea警告信息
    Spring Team recommends: "Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies".
    Spring Team建议：“在你的bean中,使用基于构造函数的依赖注入;并且使用强制依赖关系的断言”。
# 被警告的代码
```java
@Autowired
private UserService userService;
```
<!-- more -->
# Spring Team推荐的写法
```java
private UserService userService;

public UserController(UserService userService){
    this.userService = userService;
}
```
    Spring Team推荐使用构造器注入,我猜测是java变量初始化顺序的原因
    Java变量的初始化顺序为：静态变量或静态代码块–>实例变量或初始化代码块–>构造方法 - >@Autowired
# 或者使用@Resource
```java
@Resource
private UserService userService;
```
# 简析@Resource和@Autowired
## @Resource
    默认按照byName自动注入,由J2EE提供,需要导入javax.annotation.Resource
    它有两个重要的属性name和type,而Spring将@Resource注解的name属性解析为bean的名字,而type属性则解析为bean的类型
    所以,如果使用name属性,则使用byName的自动注入策略,而使用type属性时则使用byType自动注入策略
    如果name和type属性都未指定,则默认byName注入,byName未找到时,会继续采用byType注入
## @Autowired
    采用byType自动注入,由Spring提供,需要导入org.springframework.beans.factory.annotation.Autowired
    默认情况要求对象必须存在,如果需要为null,可以设置它的required=true
    如果接口存在多个实现类,我们依然可以byName自动注入:通过与@Qualifier搭配使用
    即先byType,byType匹配到多个时,再通过byName
```java
@Autowired
@Qualifier("userServiceForXXX")
private UserService userService;
```


