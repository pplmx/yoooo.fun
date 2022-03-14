---
title: Builder Pattern in Design Pattern
date: 2020-05-22T16:38:43+08:00
slug: c7150c3f1a16e2cbf090f737c9a8e6c1
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: builder pattern, design pattern, java
description: Let's begin to learn what's Builder Pattern.
---
# 建造者模式

## Overview

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。

这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

一个 Builder 类会一步一步构造最终的对象。

该 Builder 类是独立于其他对象的。

## 主要解决

主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作。

其通常由各个部分的子对象用一定的算法构成，由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。

## 何时使用

一些基本部件不会变，而其组合经常变化的时候

## 应用实例

- KFC里的可乐，薯条 ，炸鸡翅等是不变的，而其组合（套餐）是经常变化的
- Java的StringBuilder

## 使用场景

- 当一个类的构造函数参数个数超过4个，而且这些参数有些是可选的参数，考虑使用构造者模式
- 多个部件或者零件，都可以装配到一个对象中，但是产生的运行结果又相同
- 产品类非常复杂，或者产品类中调用顺序不同产生了不同的作用
- 初始化一个对象特别复杂，如使用多个构造方法，或者说有很多参数，并且都有默认值时

## 优点

- 易扩展
- 便于控制细节风险

## 实现

![Builder Pattern](/assets/builder-pattern.png)

### Computer

```java
package individual.cy.learn.pattern.creational.builder;

/**
 * @author mystic
 */
public class Computer {
    /**
     * Required
     */
    private final String cpu;
    /**
     * Required
     */
    private final String ram;
    private final String keyboard;
    private final String headset;
    private final String display;

    public Computer(ComputerBuilder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.keyboard = builder.keyboard;
        this.headset = builder.headset;
        this.display = builder.display;
    }

    public static Computer.ComputerBuilder builder(String cpu, String ram) {
        return new Computer.ComputerBuilder(cpu, ram);
    }

    public static class ComputerBuilder {
        private final String cpu;
        private final String ram;
        private String keyboard;
        private String headset;
        private String display;

        public ComputerBuilder(String cpu, String ram) {
            this.cpu = cpu;
            this.ram = ram;
        }

        public Computer build() {
            return new Computer(this);
        }

        public Computer.ComputerBuilder keyboard(String keyboard) {
            this.keyboard = keyboard;
            return this;
        }

        public Computer.ComputerBuilder headset(String headset) {
            this.headset = headset;
            return this;
        }

        public ComputerBuilder display(String display) {
            this.display = display;
            return this;
        }
    }

    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram='" + ram + '\'' +
                ", keyboard='" + keyboard + '\'' +
                ", headset='" + headset + '\'' +
                ", display='" + display + '\'' +
                '}';
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.creational.builder;

/**
 * @author mystic
 */
public class BuilderPatternTester {
    public static void main(String[] args) {
        Computer hp = Computer.builder("i9", "32G").build();
        Computer dell = Computer.builder("i9", "32G")
                .display("Samsung")
                .headset("Beats")
                .keyboard("Filco")
                .build();
        System.out.println("dell = " + dell);
        System.out.println("hp = " + hp);
    }
}
```

---

```text
16:36:50: Executing task 'BuilderPatternTester.main()'...

Configuration on demand is an incubating feature.
> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes

> Task :BuilderPatternTester.main()
dell = Computer{cpu='i9', ram='32G', keyboard='Filco', headset='Beats', display='Samsung'}
hp = Computer{cpu='i9', ram='32G', keyboard='null', headset='null', display='null'}

BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 executed
16:36:52: Task execution finished 'BuilderPatternTester.main()'.
```
