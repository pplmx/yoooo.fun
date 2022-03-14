---
title: Proxy Pattern in Design Pattern
date: 2020-05-19T15:42:27+08:00
slug: 98f5e2a74b35c88347b6eff71e0c47da
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: proxy pattern, design pattern, java
description: Let's begin to learn what's Proxy Pattern.
---
# 代理模式

## Overview

代理模式(Design Pattern)，就是给一个对象提供一个代理，并由代理对象控制对原对象的引用。

它使得客户不能直接与真正的目标对象通信。

代理对象是目标对象的代表，其他需要与这个目标对象打交道的操作都是和这个代理对象在交涉。

代理对象可以在客户端和目标对象之间起到中介的作用，这样起到了的作用和保护了目标对象的，同时也在一定程度上面减少了系统的耦合度。

## 主要解决

在直接访问对象时带来的问题。

比如说：要访问的对象在远程的机器上。

在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

## 何时使用

想在访问一个类时， 进行一些控制

## 应用实例

-   买火车票，不一定要去火车站，也可以去代售点
-   Spring AOP

## 优点

- 职责清晰
- 高扩展性
- 智能化

## 实现

![Proxy Pattern](/assets/proxy-pattern.png)

### Designer Interface

```java
public interface IDesigner {
    /**
     * some demands needed to be implemented
     *
     * @param demandName demand name
     */
    void implementsDemand(String demandName);
}
```

### Java Designer

```java
public class JavaDesigner implements IDesigner {
    private final String name;

    public JavaDesigner(String name) {
        this.name = name;
    }

    @Override
    public void implementsDemand(String demandName) {
        System.out.println("A Java Designer [ " + name + " ] implemented demand: {{ " + demandName + " }} in JAVA!");
    }
}
```

### 静态代理

```java
public class DesignerProxy implements IDesigner {
    private final IDesigner programmer;

    public DesignerProxy(IDesigner programmer) {
        this.programmer = programmer;
    }

    @Override
    public void implementsDemand(String demandName) {
        programmer.implementsDemand(demandName);
    }
}
```

```java
public class Customer {
    public static void main(String[] args) {
        IDesigner designer = new JavaDesigner("cc");
        IDesigner proxy = new DesignerProxy(designer);
        proxy.implementsDemand("Add User Management");
    }
}
```

### 动态代理

```java
public class DesignerDynamicProxy implements InvocationHandler {
    private final IDesigner designer;

    public DesignerDynamicProxy(IDesigner designer) {
        this.designer = designer;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(designer, args);
    }
}
```

```java
public class Customer {

    public static void main(String[] args) {
        // 被代理的真实对象
        IDesigner designer = new JavaDesigner("cc");
        // 创建中介类实例
        InvocationHandler handler = new DesignerDynamicProxy(designer);
        // 动态产生一个代理类
        IDesigner proxy = (IDesigner) Proxy.newProxyInstance(
                designer.getClass().getClassLoader(),
                designer.getClass().getInterfaces(),
                handler);
        // 通过代理类,执行doSomething()方法
        proxy.implementsDemand("Modify User Management");
    }

}
```
