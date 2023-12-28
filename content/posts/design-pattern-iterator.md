---
categories:
    - java
date: 2020-06-10T14:26:19Z
description: Let's begin to learn what's Iterator Pattern.
keywords: Iterator Pattern, design pattern, java
lastmod: 2020-06-10T14:33:08Z
tags:
    - general
    - design pattern
title: Iterator Pattern in Design Pattern
---



# 迭代器模式

## Overview

迭代器模式（Iterator Pattern）是 Java 和 .Net 编程环境中非常常用的设计模式。

这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。

迭代器模式属于行为型模式。

## 主要解决

不同的方式来遍历整个整合对象。

## 何时使用

遍历一个聚合对象。

## 应用实例

- JAVA 中的 iterator

## 优点

- 它支持以不同的方式遍历一个聚合对象
- 迭代器简化了聚合类
- 在同一个聚合上可以有多个遍历
- 在迭代器模式中，增加新的聚合类和迭代器类都很方便，无须修改原有代码

## 实现

![Iterator Pattern](assets/iterator-pattern.png)

### Iterator

```java
package individual.cy.learn.pattern.behavioral.iterator;

/**
 * @author mystic
 */
public interface Iterator {
    /**
     * has next()
     * @return true or false
     */
    boolean hasNext();

    /**
     * next obj
     * @return next Object
     */
    Object next();
}
```

### StringArrayIterator

```java
package individual.cy.learn.pattern.behavioral.iterator;

/**
 * @author mystic
 */
public class StringArrayIterator implements Iterator{

    private final String[] args;
    private int idx;

    public StringArrayIterator(String[] args) {
        this.args = args;
    }

    @Override
    public boolean hasNext() {
        return idx < args.length;
    }

    @Override
    public Object next() {
        if(idx < args.length){
            return args[idx++];
        }
        return null;
    }
}
```

### Container

```java
package individual.cy.learn.pattern.behavioral.iterator;

/**
 * @author mystic
 */
public interface Container {
    /**
     * get iterator
     * @return Iterator
     */
    Iterator getIterator();
}
```

### NameList

```java
package individual.cy.learn.pattern.behavioral.iterator;

/**
 * @author mystic
 */
public class NameList implements Container {

    public String[] names = {"Robert", "John", "Julie", "Lora"};

    @Override
    public Iterator getIterator() {
        return new StringArrayIterator(names);
    }

}
```

### Tester

```java
package individual.cy.learn.pattern.behavioral.iterator;

/**
 * @author mystic
 */
public class IteratorPatternDemo {
    public static void main(String[] args) {
        NameList nameList = new NameList();
        for (Iterator iter = nameList.getIterator(); iter.hasNext(); ) {
            String name = (String) iter.next();
            System.out.println("name = " + name);
        }
    }
}
```

---

```text
name = Robert
name = John
name = Julie
name = Lora
```

