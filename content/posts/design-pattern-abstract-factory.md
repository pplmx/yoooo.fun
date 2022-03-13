---
title: Abstract Factory Pattern in Design Pattern
date: 2020-05-22T13:28:40+08:00
slug: 4bac4bad3282cf57d0c4721157675b90
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: abstract factory pattern, design pattern, java
description: Let's begin to learn what's Abstract Factory Pattern.
---
# 抽象工厂模式

## Overview

抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。

这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。

每个生成的工厂都能按照`工厂模式`提供对象。

## 主要解决

主要解决接口选择的问题。

## 何时使用

系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。

## 应用实例

- QQ 换皮肤，一整套一起换

## 优点

- 当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。

## 实现

![Abstract Factory Pattern](assets/abstractfactory-pattern.png)

### Shape and Subclass

#### Shape interface

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public interface GeometricShape {
    /**
     * draw a geometric shape
     */
    void draw();
}
```

#### ShapeType2D

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public enum ShapeType2D {
    /**
     * 2d geometric shape
     */
    LINE, CIRCLE, SQUARE
}
```

#### ShapeType3D

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public enum ShapeType3D {
    /**
     * 3d geometric shape
     */
    SPHERE, CUBE, CYLINDER
}
```

#### 2D Shape

##### Line

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Line implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Line.draw");
    }
}
```

##### Circle

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Circle implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Circle.draw");
    }
}

```

##### Square

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Square implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Square.draw");
    }
}
```

#### 3D Shape

##### Cude

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Cube implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Cube.draw");
    }
}
```

##### Sphere

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Sphere implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Sphere.draw");
    }
}
```

##### Cylinder

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class Cylinder implements GeometricShape {
    @Override
    public void draw() {
        System.out.println("Cylinder.draw");
    }
}
```

### Factory

#### Abstract Factory

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public abstract class AbstractFactory {
    /**
     * To get a 2d geometric shape
     *
     * @param type shape name
     * @return Geometric Shape
     */
    public abstract GeometricShape getGeometricShape2D(ShapeType2D type);

    /**
     * To get a 3d geometric shape
     *
     * @param type shape name
     * @return Geometric Shape
     */
    public abstract GeometricShape getGeometricShape3D(ShapeType3D type);
}
```

#### Two Dimension Shape Factory

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class TwoDimensionShapeFactory extends AbstractFactory {
    @Override
    public GeometricShape getGeometricShape2D(ShapeType2D type) {
        switch (type) {
            case LINE:
                return new Line();
            case CIRCLE:
                return new Circle();
            case SQUARE:
                return new Square();
            default:
                return null;
        }
    }

    @Override
    public GeometricShape getGeometricShape3D(ShapeType3D type) {
        return null;
    }
}
```

#### Three Dimension Shape Factory

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class ThreeDimensionShapeFactory extends AbstractFactory {

    @Override
    public GeometricShape getGeometricShape2D(ShapeType2D type) {
        return null;
    }

    @Override
    public GeometricShape getGeometricShape3D(ShapeType3D type) {
        switch (type) {
            case SPHERE:
                return new Sphere();
            case CUBE:
                return new Cube();
            case CYLINDER:
                return new Cylinder();
            default:
                return null;
        }
    }
}

```

#### Factory Creator

```java
package individual.cy.learn.pattern.creational.abstractfactory;

import java.util.function.Supplier;

/**
 * @author mystic
 */
public enum FactoryCreator {
    /**
     * create 2D, 3D geometric shape
     */
    TWO_D_SHAPE_FACTORY(TwoDimensionShapeFactory::new), THREE_D_SHAPE_FACTORY(ThreeDimensionShapeFactory::new);

    private final Supplier<AbstractFactory> factorySupplier;

    FactoryCreator(Supplier<AbstractFactory> factorySupplier) {
        this.factorySupplier = factorySupplier;
    }

    public AbstractFactory getInstance() {
        return factorySupplier.get();
    }

    public static AbstractFactory getFactory(FactoryCreator creator) {
        return creator.getInstance();
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.creational.abstractfactory;

/**
 * @author mystic
 */
public class AbstractFactoryPatternTester {
    public static void main(String[] args) {
        // draw 2d shape
        FactoryCreator.TWO_D_SHAPE_FACTORY
                .getInstance()
                .getGeometricShape2D(ShapeType2D.LINE)
                .draw();

        // draw 3d shape
        FactoryCreator.THREE_D_SHAPE_FACTORY
                .getInstance()
                .getGeometricShape3D(ShapeType3D.CYLINDER)
                .draw();
    }
}
```

---

```text
13:18:52: Executing task 'AbstractFactoryPatternTester.main()'...

Configuration on demand is an incubating feature.
> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes

> Task :AbstractFactoryPatternTester.main()
Line.draw
Cylinder.draw

BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 executed
13:18:54: Task execution finished 'AbstractFactoryPatternTester.main()'.
```