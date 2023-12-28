---
categories:
    - java
date: 2020-05-21T13:34:16Z
description: Let's begin to learn what's Factory Pattern.
keywords: factory pattern, design pattern, java
lastmod: 2020-06-06T22:48:36Z
tags:
    - general
    - design pattern
title: Factory Pattern in Design Pattern
---



# 工厂模式

## Overview

工厂方法模式一种创建对象的模式，它被广泛应用在JDK中以及Spring和Struts框架中。

这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

通过给工厂对象传递不同参数来实现获得不同的子类。

## 主要解决

主要解决接口选择的问题。

我们通过工厂来替我们选择，对于不知情的人，只要传入参数，工厂会自动为我们选择一个类。

## 何时使用

我们明确地计划不同条件下创建不同实例时。

## 应用实例

- 购买汽车，直接去工厂提货，而不需要知道汽车是怎么制造的
- Hibernate 换数据库只需换方言和驱动就可以

## 优点

- 一个调用者想创建一个对象，只要知道其名称就可以了
- 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以
- 屏蔽产品的具体实现，调用者只关心产品的接口

## 实现

![Factory Pattern](assets/factory-pattern.png)

### Shape and Subclass

#### Shape interface

```java
public interface Shape {
    /**
     * draw a geometry
     */
    void draw();
}
```

#### Circle

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Circle.draw");
    }
}
```

#### Rectangle

```java
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Rectangle.draw");
    }
}
```

#### Square

```java
public class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Square.draw");
    }
}
```

### Factory

#### Solution 1

```java
public class ShapeFactory {
    public Shape getShape(String shapeType) {
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("circle")) {
            return new Circle();
        } else if (shapeType.equalsIgnoreCase("rectangle")) {
            return new Rectangle();
        } else if (shapeType.equalsIgnoreCase("square")) {
            return new Square();
        }
        return null;
    }
}
```

#### Solution 2

```java
public enum ShapeCreator {
    /**
     * create Circle, Rectangle, Square
     */
    CIRCLE(Circle::new), RECTANGLE(Rectangle::new), SQUARE(Square::new);

    private final Supplier<Shape> shapeSupplier;

    ShapeCreator(Supplier<Shape> shapeSupplier) {
        this.shapeSupplier = shapeSupplier;
    }

    public Shape getInstance() {
        return shapeSupplier.get();
    }

    public static Shape getShape(ShapeCreator creator) {
        return creator.getInstance();
    }
}
```

#### Solution 3

```java
public enum ShapeCreator2 {
    /**
     * create Circle, Rectangle, Square
     */
    CIRCLE(new Circle()), RECTANGLE(new Rectangle()), SQUARE(new Square());

    private final Shape shape;

    ShapeCreator2(Shape shape) {
        this.shape = shape;
    }

    public Shape getShape() {
        return shape;
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.creational.factory;

/**
 * @author mystic
 */
public class FactoryPatternTester {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();

        // Acquire Circle
        Shape circle = factory.getShape("circle");
        circle.draw();

        // Acquire Rectangle
        Shape rectangle = factory.getShape("rectangle");
        rectangle.draw();

        // Acquire Square
        Shape square = factory.getShape("square");
        square.draw();

        System.out.println();

        ShapeCreator.CIRCLE.getInstance().draw();
        ShapeCreator.RECTANGLE.getInstance().draw();
        ShapeCreator.SQUARE.getInstance().draw();

        System.out.println();

        System.out.println(ShapeCreator.CIRCLE.getInstance());
        System.out.println(ShapeCreator.CIRCLE.getInstance());
        System.out.println(ShapeCreator.CIRCLE.getInstance());
        // Or this usage
        System.out.println(ShapeCreator.getShape(ShapeCreator.CIRCLE));
        System.out.println(ShapeCreator.getShape(ShapeCreator.CIRCLE));
        System.out.println(ShapeCreator.getShape(ShapeCreator.CIRCLE));

        System.out.println();

        System.out.println(ShapeCreator2.CIRCLE.getShape());
        System.out.println(ShapeCreator2.CIRCLE.getShape());
        System.out.println(ShapeCreator2.CIRCLE.getShape());

    }
}
```

---

```text
Circle.draw
Rectangle.draw
Square.draw

Circle.draw
Rectangle.draw
Square.draw

individual.cy.learn.pattern.creational.factory.Circle@12f40c25
individual.cy.learn.pattern.creational.factory.Circle@3ada9e37
individual.cy.learn.pattern.creational.factory.Circle@5cbc508c
individual.cy.learn.pattern.creational.factory.Circle@3419866c
individual.cy.learn.pattern.creational.factory.Circle@63e31ee
individual.cy.learn.pattern.creational.factory.Circle@68fb2c38

individual.cy.learn.pattern.creational.factory.Circle@2eafffde
individual.cy.learn.pattern.creational.factory.Circle@2eafffde
individual.cy.learn.pattern.creational.factory.Circle@2eafffde

```

ShapeCreator创建子对象,都是不同的

ShapeCreator2子对象都是同一个


