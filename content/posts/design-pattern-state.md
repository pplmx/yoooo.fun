---
title: State Pattern in Design Pattern
date: 2020-06-12T16:49:39+08:00
slug: 21c155711c41eddfff250855d401a8e4
draft: false
lastmod: 2020-06-12T16:55:45+08:00
categories: [java]
tags: [general, design pattern]
keywords: state pattern, design pattern, java
description: Let's begin to learn what's State Pattern.
---
# 状态模式

## Overview

在状态模式（State Pattern）中，类的行为是基于它的状态改变的。

这种类型的设计模式属于行为型模式。

在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象。

## 主要解决

对象的行为依赖于它的状态（属性），并且可以根据它的状态改变而改变它的相关行为

## 何时使用

代码中包含大量与对象状态有关的条件语句

## 应用实例

- 购物订单的状态改变(未付款, 付款, 确认收货...)

## 优点

- 封装了转换规则
- 枚举可能的状态，在枚举状态之前需要确定状态种类
- 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为
- 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块
- 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数

## 实现

![State Pattern](/assets/state-pattern.png)

### Package State

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public interface PackageState {
    /**
     * update package state
     * @param ctx context
     */
    void updateState(DeliveryContext ctx);
}
```

#### Acknowledged

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class Acknowledged implements PackageState {

    private static volatile Acknowledged singleton = null;

    private Acknowledged() {
    }

    public static Acknowledged instance() {
        if (singleton == null) {
            synchronized (Acknowledged.class) {
                if (singleton == null) {
                    singleton = new Acknowledged();
                }
            }
        }
        return singleton;
    }


    @Override
    public void updateState(DeliveryContext ctx) {
        System.out.println("Package is acknowledged !!");
        ctx.setCurrentState(Shipped.instance());
    }
}
```

#### Shipped

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class Shipped implements PackageState {

    private static volatile Shipped singleton = null;

    private Shipped() {
    }

    public static Shipped instance() {
        if (singleton == null) {
            synchronized (Shipped.class) {
                if (singleton == null) {
                    singleton = new Shipped();
                }
            }
        }
        return singleton;
    }

    @Override
    public void updateState(DeliveryContext ctx) {
        System.out.println("Package is shipped !!");
        ctx.setCurrentState(InTransition.instance());
    }
}
```

#### In Transition

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class InTransition implements PackageState {

    private static volatile InTransition singleton = null;

    private InTransition() {
    }

    public static InTransition instance() {
        if (singleton == null) {
            synchronized (InTransition.class) {
                if (singleton == null) {
                    singleton = new InTransition();
                }
            }
        }
        return singleton;
    }

    @Override
    public void updateState(DeliveryContext ctx) {
        System.out.println("Package is in transition !!");
        ctx.setCurrentState(OutForDelivery.instance());
    }
}
```

#### Out of Delivery

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class OutForDelivery implements PackageState {

    private static volatile OutForDelivery singleton = null;

    private OutForDelivery() {
    }

    public static OutForDelivery instance() {
        if (singleton == null) {
            synchronized (OutForDelivery.class) {
                if (singleton == null) {
                    singleton = new OutForDelivery();
                }
            }
        }
        return singleton;
    }

    @Override
    public void updateState(DeliveryContext ctx) {
        System.out.println("Package is out of delivery !!");
        ctx.setCurrentState(Delivered.instance());
    }
}
```

#### Delivered

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class Delivered implements PackageState {

    private static volatile Delivered singleton = null;

    private Delivered() {
    }

    public static Delivered instance() {
        if (singleton == null) {
            synchronized (Delivered.class) {
                if (singleton == null) {
                    singleton = new Delivered();
                }
            }
        }
        return singleton;
    }

    @Override
    public void updateState(DeliveryContext ctx) {
        System.out.println("Package is delivered!!");
    }
}
```

### DeliveryContext

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class DeliveryContext {
    private PackageState currentState;
    private String packageId;

    public DeliveryContext(PackageState currentState, String packageId) {
        this.currentState = currentState != null ? currentState : Acknowledged.instance();
        this.packageId = packageId;
    }

    public void update() {
        currentState.updateState(this);
    }

    public PackageState getCurrentState() {
        return currentState;
    }

    public void setCurrentState(PackageState currentState) {
        this.currentState = currentState;
    }

    public String getPackageId() {
        return packageId;
    }

    public void setPackageId(String packageId) {
        this.packageId = packageId;
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.state;

/**
 * @author mystic
 */
public class StatePatternTester {
    public static void main(String[] args) {
        DeliveryContext ctx = new DeliveryContext(null, "Test1");
        ctx.update();
        ctx.update();
        ctx.update();
        ctx.update();
        ctx.update();
    }
}
```

---

```text
Package is acknowledged !!
Package is shipped !!
Package is in transition !!
Package is out of delivery !!
Package is delivered !!
```
