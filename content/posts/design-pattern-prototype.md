---
title: Prototype Pattern in Design Pattern
date: 2020-06-10T17:20:16+08:00
slug: 59c60e86bc99f837b14dfe572ea0efec
draft: false
lastmod: 2020-06-10T17:22:33+08:00
categories: [java]
tags: [general, design pattern]
keywords: prototype pattern, design pattern, java
description: Let's begin to learn what's Prototype Pattern.
---
# 原型模式

## Overview

原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。

这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。

当直接创建对象的代价比较大时，则采用这种模式。

例如，一个对象需要在一个高代价的数据库操作之后被创建。

我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。

## 主要解决

在运行期建立和删除原型。

## 何时使用

- 当一个系统应该独立于它的产品创建，构成和表示时
- 当要实例化的类是在运行时刻指定时，例如，通过动态装载
- 为了避免创建一个与产品类层次平行的工厂类层次时
- 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。

## 应用实例

- 细胞分裂
- JAVA 中的 Object clone() 方法

## 优点

- 性能提高
- 逃避构造函数的约束

## 实现

![Prototype Pattern](assets/prototype-pattern.png)

### BaseShape

```java
package individual.cy.learn.pattern.creational.prototype;

/**
 * @author mystic
 */
public abstract class BaseShape implements Cloneable {
    private String id;
    protected String type;

    /**
     * to draw a geometric
     */
    public abstract void draw();

    public String getType() {
        return type;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    @Override
    public BaseShape clone() {
        BaseShape clone = null;
        try {
            clone = (BaseShape) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }

}
```

#### Circle

```java
package individual.cy.learn.pattern.creational.prototype;

/**
 * @author mystic
 */
public class Circle extends BaseShape {

    public Circle() {
        type = "Circle";
    }

    @Override
    public void draw() {
        System.out.println("Circle.draw");
    }
}
```

#### Square

```java
package individual.cy.learn.pattern.creational.prototype;

/**
 * @author mystic
 */
public class Square extends BaseShape {

    public Square() {
        type = "Square";
    }

    @Override
    public void draw() {
        System.out.println("Square.draw");
    }
}
```

#### Rectangle

```java
package individual.cy.learn.pattern.creational.prototype;

/**
 * @author mystic
 */
public class Rectangle extends BaseShape {

    public Rectangle() {
        type = "Rectangle";
    }

    @Override
    public void draw() {
        System.out.println("Rectangle.draw");
    }
}
```

### ShapeCache

```java
package individual.cy.learn.pattern.creational.prototype;

import java.util.Hashtable;

/**
 * @author mystic
 */
public class ShapeCache {

    private static final Hashtable<String, BaseShape> SHAPE_MAP = new Hashtable<String, BaseShape>();

    public static BaseShape getShape(String shapeId) {
        BaseShape cachedShape = SHAPE_MAP.get(shapeId);
        return cachedShape.clone();
    }

    public static void loadCache() {
        Circle circle = new Circle();
        circle.setId("1");
        SHAPE_MAP.put(circle.getId(), circle);

        Square square = new Square();
        square.setId("2");
        SHAPE_MAP.put(square.getId(), square);

        Rectangle rectangle = new Rectangle();
        rectangle.setId("3");
        SHAPE_MAP.put(rectangle.getId(), rectangle);
    }

}
```

### Tester

```java
package individual.cy.learn.pattern.creational.prototype;

/**
 * @author mystic
 */
public class PrototypePatternTester {
    public static void main(String[] args) {
        ShapeCache.loadCache();

        BaseShape cachedShape1 = ShapeCache.getShape("1");
        System.out.println("cachedShape1.getType() = " + cachedShape1.getType());

        BaseShape cachedShape2 = ShapeCache.getShape("2");
        System.out.println("cachedShape2.getType() = " + cachedShape2.getType());

        BaseShape cachedShape3 = ShapeCache.getShape("3");
        System.out.println("cachedShape3.getType() = " + cachedShape3.getType());
    }
}
```

---

```text
cachedShape1.getType() = Circle
cachedShape2.getType() = Square
cachedShape3.getType() = Rectangle
```