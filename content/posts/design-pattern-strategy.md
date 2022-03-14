---
title: Strategy Pattern in Design Pattern
date: 2020-05-31T20:27:11+08:00
slug: 160d8b999b1724c549f454fcbfa9d350
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: strategy pattern, design pattern, java
description: Let's begin to learn what's strategy Pattern.
---
# 策略模式

## Overview

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。

这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。

策略对象改变 context 对象的执行算法。

## 主要解决

在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。


## 何时使用

一个系统有许多许多类，而区分它们的只是他们直接的行为。

## 应用实例

- 诸葛亮的锦囊妙计，每一个锦囊就是一个策略
- 旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略
- JAVA AWT 中的 LayoutManager


## 优点

- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好

## 注意事项

如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

## 实现

![Strategy Pattern](/assets/strategy-pattern.png)

### Behavior

#### Jump Interface

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public interface JumpBehavior{
    /**
     * jump
     */
    void jump();
}
```

#### Short Jump

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class ShortJump implements JumpBehavior {
    @Override
    public void jump() {
        System.out.println("ShortJump.jump");
    }
}
```

#### Long Jump

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class LongJump implements JumpBehavior {
    @Override
    public void jump() {
        System.out.println("LongJump.jump");
    }
}
```

#### Kick Interface

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public interface KickBehavior{
    /**
     * kick
     */
    void kick();
}
```

#### Lightning Kick

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class LightningKick implements KickBehavior {
    @Override
    public void kick() {
        System.out.println("LightningKick.kick");
    }
}
```

#### Tornado Kick

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class TornadoKick implements KickBehavior {
    @Override
    public void kick() {
        System.out.println("TornadoKick.kick");
    }
}
```

### Fighter

#### Fighter Interface

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public abstract class BaseFighter {
    protected KickBehavior kickBehavior;
    protected JumpBehavior jumpBehavior;

    public BaseFighter(KickBehavior kickBehavior, JumpBehavior jumpBehavior) {
        this.kickBehavior = kickBehavior;
        this.jumpBehavior = jumpBehavior;
    }

    /**
     * display
     */
    public abstract void display();

    public void punch() {
        System.out.println("BaseFighter.punch");
    }

    public void roll() {
        System.out.println("BaseFighter.roll");
    }

    public void kick() {
        // delegate to KickBehavior
        kickBehavior.kick();
    }

    public void jump() {
        // delegate to JumpBehavior
        jumpBehavior.jump();
    }

    public void setKickBehavior(KickBehavior kickBehavior) {
        this.kickBehavior = kickBehavior;
    }

    public void setJumpBehavior(JumpBehavior jumpBehavior) {
        this.jumpBehavior = jumpBehavior;
    }
}
```

#### YeWen Fighter

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class YeWenFighter extends BaseFighter {
    public YeWenFighter(KickBehavior kickBehavior, JumpBehavior jumpBehavior) {
        super(kickBehavior, jumpBehavior);
    }

    @Override
    public void display() {
        System.out.println("YeWenFighter.display");
    }
}
```

#### Ken Fighter

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class KenFighter extends BaseFighter {
    public KenFighter(KickBehavior kickBehavior, JumpBehavior jumpBehavior) {
        super(kickBehavior, jumpBehavior);
    }

    @Override
    public void display() {
        System.out.println("KenFighter.display");
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.strategy;

/**
 * @author mystic
 */
public class StrategyPatternTester {
    public static void main(String[] args) {
        // make some behaviors first
        JumpBehavior shortJump = new ShortJump();
        JumpBehavior longJump = new LongJump();
        KickBehavior tornadoKick = new TornadoKick();

        // Make a fighter with desired behaviors
        BaseFighter ken = new KenFighter(tornadoKick, shortJump);
        ken.display();

        // Test behaviors
        ken.punch();
        ken.roll();
        ken.jump();
        ken.kick();

        // change behavior dynamically
        ken.setJumpBehavior(longJump);
        ken.jump();

        ken.setKickBehavior(tornadoKick);
        ken.kick();
    }
}
```
