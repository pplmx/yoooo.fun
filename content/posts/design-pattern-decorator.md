---
title: Decorator Pattern in Design Pattern
date: 2020-05-24T21:12:03+08:00
slug: 2ea5649cdc51e91b7274d87de7e73b0d
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: decorator pattern, design pattern, java
description: Let's begin to learn what's Decorator Pattern.
---
# 装饰器模式

## Overview

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。

这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

## 主要解决

一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

## 何时使用

在不想增加很多子类的情况下扩展类。

## 应用实例

- 给画，添加上画框

## 优点

- 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。

## 实现

![Decorator Pattern](/assets/decorator-pattern.png)

### Shape

#### Shape Interface

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public interface Shape {
    /**
     * to draw a geometric shape
     */
    void draw();
}
```

#### Circle

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Circle.draw");
    }
}
```

#### Rectangle

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Rectangle.draw");
    }
}
```

### Decorator

#### AbstractShapeDecorator

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public abstract class AbstractShapeDecorator implements Shape {
    protected Shape decoratedShape;

    public AbstractShapeDecorator(Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    @Override
    public void draw() {
        decoratedShape.draw();
    }
}
```

#### RedShapeDecorator

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public class RedShapeDecorator extends AbstractShapeDecorator {
    public RedShapeDecorator(Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    public void draw() {
        super.draw();
        setRedBorder(decoratedShape);
    }

    private void setRedBorder(Shape decoratedShape) {
        System.out.println("RedShapeDecorator.setRedBorder");
    }
}
```

#### VioletShapeDecorator

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public class VioletShapeDecorator extends AbstractShapeDecorator {
    public VioletShapeDecorator(Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    public void draw() {
        super.draw();
        setVioletBorder(decoratedShape);
    }

    public void setVioletBorder(Shape decoratedShape) {
        System.out.println("VioletShapeDecorator.setVioletBorder");
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.structural.decorator;

/**
 * @author mystic
 */
public class DecoratorPatternTester {
    public static void main(String[] args) {
        Shape circle = new Circle();
        Shape redCircle = new RedShapeDecorator(new Circle());
        Shape redRectangle = new RedShapeDecorator(new Rectangle());
        Shape violetCircle = new VioletShapeDecorator(new Circle());
        Shape violetRectangle = new VioletShapeDecorator(new Rectangle());
//        Shape redCircle = new RedShapeDecorator(Circle::new);
//        Shape redRectangle = new RedShapeDecorator(Rectangle::new);
//        Shape violetCircle = new VioletShapeDecorator(Circle::new);
//        Shape violetRectangle = new VioletShapeDecorator(Rectangle::new);
        circle.draw();
        redCircle.draw();
        redRectangle.draw();
        violetCircle.draw();
        violetRectangle.draw();
    }
}
```

---

```text
Circle.draw
Circle.draw
RedShapeDecorator.setRedBorder
Rectangle.draw
RedShapeDecorator.setRedBorder
Circle.draw
VioletShapeDecorator.setVioletBorder
Rectangle.draw
VioletShapeDecorator.setVioletBorder
```

如果执行注释的代码,即使用`::new`创建对象,则输出

```text
Circle.draw
RedShapeDecorator.setRedBorder
RedShapeDecorator.setRedBorder
VioletShapeDecorator.setVioletBorder
VioletShapeDecorator.setVioletBorder
```
