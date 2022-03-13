---
title: Let's learn what's singleton
date: 2020-04-30T16:12:37+08:00
lastmod: 2020-04-30T16:17:43+08:00
slug: 2ed500a3529637175e675a8791b7c56e
draft: false
categories: [java]
tags: [design pattern]
keywords: singleton, design pattern, java
description: Let's learn what's singleton
---
# 单例(Singleton)

数学与逻辑学中，singleton定义为“有且仅有一个元素的集合”。

## 什么是单例?

一个类有且仅有一个实例, 并提供一个可以访问它的全局访问点.

## 单例有什么用?

解决一个全局使用类的**频繁创建**与**销毁**.

## 怎么实现单例?

### 实现方案

-   隐藏类的构造方法
-   定义一个公有的静态方法, 通过它返回类的唯一实例

### java实现

1.  DCL(Double Checked Lock)实现

```java
public class Lazy4SafeDoubleCheck {
    private static volatile Lazy4SafeDoubleCheck singleton = null;

    private Lazy4SafeDoubleCheck() {
        // 1. 避免反射创建多个实例
        if(singleton != null){
            throw new RuntimeException()
        }
    }

    public static Lazy4SafeDoubleCheck getSingleton() {
        if (singleton == null) {
            synchronized (Lazy4SafeDoubleCheck.class) {
                if (singleton == null) {
                    singleton = new Lazy4SafeDoubleCheck();
                }
            }
        }
        return singleton;
    }
    
    // 2. 避免反序列化创建多个实例
    private Object readResolve() throws ObjectStreamException {
    	return singleton;
    }
}
```

2.  枚举实现

```java
public enum SingletonEnum {
    /**
     * 使用枚举实现的单例
     */
    INSTANCE;

    public void otherMethods() {
        System.out.println("Something");
    }
}
```