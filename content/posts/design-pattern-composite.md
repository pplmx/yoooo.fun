---
title: Composite Pattern in Design Pattern
date: 2020-06-17T18:08:39+08:00
lastmod: 2020-06-17T18:10:28+08:00
slug: eae0d7ae830b5aa84d5bbda7c8a5c80c
draft: false
categories: [java]
tags: [general, design pattern]
keywords: Composite pattern, design pattern, java
description: Let's begin to learn what's Composite Pattern.
---
# 组合模式

## Overview

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。

组合模式依据树形结构来组合对象，用来表示部分以及整体层次。

这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

这种模式创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式。

## 主要解决

它在我们树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以像处理简单元素一样来处理复杂元素，从而使得客户程序与复杂元素的内部结构解耦。

## 何时使用

- 您想表示对象的部分-整体层次结构（树形结构）
- 您希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象

## 应用实例

- 算术表达式包括操作数、操作符和另一个操作数，其中，另一个操作符也可以是操作数、操作符和另一个操作数

## 优点

- 高层模块调用简单
- 节点自由增加

## 实现

![Composite Pattern](assets/composite-pattern.png)

### Employee

```java
package individual.cy.learn.pattern.structural.composite;

import java.util.ArrayList;
import java.util.List;

/**
 * @author mystic
 */
public class Employee {
    private final int id;
    private String name;
    private String dept;
    private int salary;
    private final List<Employee> subordinateList;

    public Employee(int id, String name, String dept, int salary) {
        this.id = id;
        this.name = name;
        this.dept = dept;
        this.salary = salary;
        this.subordinateList = new ArrayList<>();
    }

    public void add(Employee employee) {
        subordinateList.add(employee);
    }

    public void remove(Employee employee) {
        subordinateList.remove(employee);
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDept() {
        return dept;
    }

    public void setDept(String dept) {
        this.dept = dept;
    }

    public int getSalary() {
        return salary;
    }

    public void setSalary(int salary) {
        this.salary = salary;
    }

    public List<Employee> getSubordinateList() {
        return subordinateList;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", dept='" + dept + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.structural.composite;

/**
 * @author mystic
 */
public class CompositePatternTester {
    public static void main(String[] args) {
        Employee ceo = new Employee(10000, "Adam", "CEO", 70000);

        Employee headSales = new Employee(20000, "Robert", "Head Sales", 20000);
        Employee headMarketing = new Employee(30000, "Michel", "Head Marketing", 20000);

        Employee salesExecutive1 = new Employee(20001, "Richard", "Sales", 10000);
        Employee salesExecutive2 = new Employee(20002, "Rob", "Sales", 10000);

        Employee clerk1 = new Employee(30001, "Laura", "Marketing", 10000);
        Employee clerk2 = new Employee(30002, "Bob", "Marketing", 10000);

        ceo.add(headSales);
        ceo.add(headMarketing);

        headSales.add(salesExecutive1);
        headSales.add(salesExecutive2);

        headMarketing.add(clerk1);
        headMarketing.add(clerk2);

        System.out.println("ceo = " + ceo);

        System.out.println();

        // print ceo immediate subordinate
        ceo.getSubordinateList().forEach(System.out::println);

        System.out.println();

        // print ceo immediate subordinate's immediate subordinate
        ceo.getSubordinateList().stream()
                .flatMap(ceoSubordinate -> ceoSubordinate.getSubordinateList().stream())
                .forEach(System.out::println);
    }
}
```

---

```text
ceo = Employee{id=10000, name='Adam', dept='CEO', salary=70000}

Employee{id=20000, name='Robert', dept='Head Sales', salary=20000}
Employee{id=30000, name='Michel', dept='Head Marketing', salary=20000}

Employee{id=20001, name='Richard', dept='Sales', salary=10000}
Employee{id=20002, name='Rob', dept='Sales', salary=10000}
Employee{id=30001, name='Laura', dept='Marketing', salary=10000}
Employee{id=30002, name='Bob', dept='Marketing', salary=10000}
```