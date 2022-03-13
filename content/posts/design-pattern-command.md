---
title: Command Pattern in Design Pattern
date: 2020-06-15T13:57:46+08:00
slug: 63380428c5c51bec051a13c88b470685
draft: false
lastmod: 2020-06-15T14:00:07+08:00
categories: [java]
tags: [general, design pattern]
keywords: command pattern, design pattern, java
description: Let's begin to learn what's Command Pattern.
---
# 命令模式

## Overview

命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。

请求以命令的形式包裹在对象中，并传给调用对象。

调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

## 主要解决

在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。

## 何时使用

在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。

在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

## 应用实例

- struts中的action核心控制器ActionServlet

## 优点

- 降低了系统耦合度
- 新的命令可以很容易添加到系统中去

## 实现

![Command Pattern](assets/command-pattern.png)

### Command

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public interface Command {
    /**
     * execute a action
     */
    void execute();
}
```

### Light

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class Light {
    public void on() {
        System.out.println("Light.on");
    }

    public void off() {
        System.out.println("Light.off");
    }
}
```

#### LightOnCommand

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class LightOnCommand implements Command {

    private final Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }
}
```

#### LightOffCommand

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class LightOffCommand implements Command {

    private final Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }
}
```

### Stereo

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class Stereo {
    public void on() {
        System.out.println("Stereo.on");
    }

    public void off() {
        System.out.println("Stereo.off");
    }

    public void setCD() {
        System.out.println("Stereo.setCD");
    }

    public void setDVD() {
        System.out.println("Stereo.setDVD");
    }

    public void setRadio() {
        System.out.println("Stereo.setRadio");
    }

    public void setVolume(int volume) {
        System.out.println("Stereo.setVolume: volume = " + volume);
    }
}
```

#### StereoOnWithCdCommand

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class StereoOnWithCdCommand implements Command {
    private final Stereo stereo;

    public StereoOnWithCdCommand(Stereo stereo) {
        this.stereo = stereo;
    }

    @Override
    public void execute() {
        stereo.on();
        stereo.setCD();
        stereo.setVolume(11);
    }
}
```

#### StereoOffCommand

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class StereoOffCommand implements Command {
    private final Stereo stereo;

    public StereoOffCommand(Stereo stereo) {
        this.stereo = stereo;
    }

    @Override
    public void execute() {
        stereo.off();
    }
}
```

### SimpleRemoteControl

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class SimpleRemoteControl {
    private Command slot;

    public void setCommand(Command command) {
        slot = command;
    }

    public void buttonWasPressed() {
        slot.execute();
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.command;

/**
 * @author mystic
 */
public class CommandPatternTester {
    public static void main(String[] args) {
        SimpleRemoteControl remote = new SimpleRemoteControl();
        Light light = new Light();
        Stereo stereo = new Stereo();

        // change command dynamically
        remote.setCommand(new LightOnCommand(light));
        remote.buttonWasPressed();

        remote.setCommand(new StereoOnWithCdCommand(stereo));
        remote.buttonWasPressed();

        remote.setCommand(new StereoOffCommand(stereo));
        remote.buttonWasPressed();
    }
}
```

---

```text
Light.on
Stereo.on
Stereo.setCD
Stereo.setVolume: volume = 11
Stereo.off
```