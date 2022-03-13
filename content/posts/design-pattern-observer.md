---
title: Observer Pattern in Design Pattern
date: 2020-06-01T16:12:19+08:00
lastmod: 2020-06-01T16:30:23+08:00
slug: a8777c0d2adcfd80991cf81402651e5c
draft: false
categories: [java]
tags: [general, design pattern]
keywords: observer pattern, design pattern, java
description: Let's begin to learn what's Observer Pattern.
---
# 观察者模式

## Overview

当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。

比如，当一个对象被修改时，则会自动通知依赖它的对象。

观察者模式属于行为型模式。

## 主要解决

一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

## 何时使用

一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。

## 应用实例

- 拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价

## 优点

- 观察者和被观察者是抽象耦合的
- 建立一套触发机制

## 实现一

![Observer Pattern](assets/observer-pattern1.png)

### Observer

#### Observer Interface

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

/**
 * @author mystic
 */
public interface Observer {
    /**
     * update
     * @param event event
     */
    void update(String event);
}
```

#### People Daily

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

/**
 * @author mystic
 */
public class PeopleDaily implements Observer {
    @Override
    public void update(String event) {
        System.out.println("Breaking news in People Daily! event = " + event);
    }
}
```

#### New York Times

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

/**
 * @author mystic
 */
public class NewYorkTimes implements Observer {
    @Override
    public void update(String event) {
        System.out.println("Breaking news in New York Times! event = " + event);
    }
}
```

### Subject

#### Subject Interface

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

/**
 * @author mystic
 */
public interface Subject {
    /**
     * register
     *
     * @param observer observer
     */
    void registerObserver(Observer observer);

    /**
     * unregister
     *
     * @param observer observer
     */
    void unregisterObserver(Observer observer);

    /**
     * notify all observers
     * @param event event
     */
    void notifyObservers(String event);
}
```

#### Feed

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

import java.util.ArrayList;
import java.util.List;

/**
 * @author mystic
 */
public class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void unregisterObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String event) {
        observers.forEach(o -> o.update(event));
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.observer.sln2;

/**
 * @author mystic
 */
public class Tester {
    public static void main(String[] args) {
        Feed globalEvent = new Feed();

        Observer newYorkTimes = new NewYorkTimes();
        Observer peopleDaily = new PeopleDaily();

        globalEvent.registerObserver(newYorkTimes);
        globalEvent.registerObserver(peopleDaily);

        globalEvent.notifyObservers("Violent Parade in USA");

        System.out.println("Unregister New York Times!");
        globalEvent.unregisterObserver(newYorkTimes);
        globalEvent.notifyObservers("Violent Parade in Canada");
    }
}
```

---

```text
Breaking news in New York Times! event = Violent Parade in USA
Breaking news in People Daily! event = Violent Parade in USA
Unregister New York Times!
Breaking news in People Daily! event = Violent Parade in Canada
```

## 实现二

![Observer Pattern](assets/observer-pattern2.png)

### Observer

#### Abstract Observer

```java
package individual.cy.learn.pattern.behavioral.observer;

/**
 * @author mystic
 */
public abstract class AbstractObserver {
    protected Subject subject;

    /**
     * update
     */
    public abstract void update();
}

```

#### Binary Observer

```java
package individual.cy.learn.pattern.behavioral.observer;

/**
 * @author mystic
 */
public class BinaryObserver extends AbstractObserver {
    public BinaryObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("BinaryObserver.update: " + Integer.toBinaryString(subject.getState()));
    }
}
```

#### Octal Observer

```java
package individual.cy.learn.pattern.behavioral.observer;

/**
 * @author mystic
 */
public class OctalObserver extends AbstractObserver {

    public OctalObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("OctalObserver.update: " + Integer.toOctalString(subject.getState()));
    }
}
```

#### Hex Observer

```java
package individual.cy.learn.pattern.behavioral.observer;

/**
 * @author mystic
 */
public class HexObserver extends AbstractObserver {
    public HexObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("HexObserver.update: " + Integer.toHexString(subject.getState()).toUpperCase());
    }
}
```

### Subject

```java
package individual.cy.learn.pattern.behavioral.observer;

import java.util.ArrayList;
import java.util.List;

/**
 * @author mystic
 */
public class Subject {
    private final List<AbstractObserver> observerList = new ArrayList<>();

    private int state;

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObservers();
    }

    public void attach(AbstractObserver observer) {
        observerList.add(observer);
    }

    public void notifyAllObservers() {
        observerList.forEach(AbstractObserver::update);
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.observer;

/**
 * @author mystic
 */
public class ObserverPatternTester {
    public static void main(String[] args) {
        Subject subject = new Subject();
        new HexObserver(subject);
        new OctalObserver(subject);
        new BinaryObserver(subject);

        System.out.println("First state change: 15");
        subject.setState(15);
        System.out.println("========================");
        System.out.println("Second state change: 10");
        subject.setState(10);
    }
}

```

---

```text
First state change: 15
HexObserver.update: F
OctalObserver.update: 17
BinaryObserver.update: 1111
========================
Second state change: 10
HexObserver.update: A
OctalObserver.update: 12
BinaryObserver.update: 1010
```