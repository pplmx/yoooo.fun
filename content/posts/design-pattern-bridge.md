---
title: Bridge Pattern in Design Pattern
date: 2020-06-10T18:24:49+08:00
lastmod: 2020-06-10T18:27:22+08:00
slug: a6e1761a2f6d06b4ecaaa7ce50659bf1
draft: false
categories: [java]
tags: [general, design pattern]
keywords: bridge pattern, design pattern, java
description: Let's begin to learn what's Bridge Pattern.
---
# 桥接模式

## Overview

桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。

这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。

这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。

这两种类型的类可被结构化改变而互不影响。

## 主要解决

在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活

## 何时使用

实现系统可能有多个角度分类，每一种角度都可能变化

## 应用实例

-   墙上的开关，可以看到的开关是抽象的，不用管里面具体怎么实现的

## 优点

- 抽象和实现的分离
- 优秀的扩展能力
- 实现细节对客户透明

## 实现

![Bridge Pattern](assets/bridge-pattern.png)

### DrawApi

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public interface DrawApi {
    /**
     * To draw a circle
     * @param radius radius
     * @param x Abscissa, X-axis
     * @param y ordinate, Y-axis
     */
    void drawCircle(int radius, int x, int y);
}
```

#### RedCircle

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public class RedCircle implements DrawApi {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("[Draw a red circle] radius = " + radius + ", x = " + x + ", y = " + y);
    }
}
```

#### PurpleCircle

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public class PurpleCircle implements DrawApi {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("[Draw a purple circle] radius = " + radius + ", x = " + x + ", y = " + y);
    }
}
```

### BaseShape

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public abstract class BaseShape {
    protected DrawApi drawApi;

    protected BaseShape(DrawApi drawApi) {
        this.drawApi = drawApi;
    }

    /**
     * To draw a geometric shape
     */
    public abstract void draw();
}
```

#### Circle

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public class Circle extends BaseShape {
    private final int x;
    private final int y;
    private final int radius;

    public Circle(DrawApi drawApi, int x, int y, int radius) {
        super(drawApi);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    @Override
    public void draw() {
        drawApi.drawCircle(radius, x, y);
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.structural.bridge;

/**
 * @author mystic
 */
public class BridgePatternTester {
    public static void main(String[] args) {
        BaseShape redCircle = new Circle(new RedCircle(), 0, 0, 3);
        BaseShape purpleCircle = new Circle(new PurpleCircle(), 0, 6, 3);

        redCircle.draw();
        purpleCircle.draw();
    }
}
```

---

```text
[Draw a red circle] radius = 3, x = 0, y = 0
[Draw a purple circle] radius = 3, x = 0, y = 6
```
