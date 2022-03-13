---
title: Compile-time and Runtime Constants
date: 2020-05-16T21:32:52+08:00
slug: d626f04a53f10244a23f70d793610123
draft: false
lastmod: 2020-05-16T22:25:38+08:00
categories: [java]
tags: [compile,runtime]
keywords: constants, runtime, compile, compile time
description: What's compile-time and runtime constants in Java?
---
# Please learn from the following code first
```java
package individual.cy.learn.mess;

/**
 * @author mystic
 */
public class CompileAndRuntimeConstant {
    public static void main(String[] args) {
        // No load static block
        System.out.println(Test.NAME);
        // No load static block
        System.out.println(Test.SCORE);
        // No load static block
        System.out.println(Test.PI);
        // load static block
        System.out.println(Test.CC);
        // load static block(cc and age only load once totally)
        System.out.println(Test.age);
        System.out.println();
    }
}

class Test {
    /**
     * The following three: compile-time constants
     */
    public static final String NAME = "shanxi";
    public static final int SCORE = 85;
    public static final double PI = 3.1415;

    /**
     * The following two: runtime-constants
     */
    public static final Integer CC = 2;
    public static int age = 23;

    static {
        System.out.println("Test static block: " + PI);
    }

    /**
     * The following one: compile-time constant
     */
    private final int gg = 10;
    private int hh = gg;

    public int getHh() {
        return hh;
    }

    public void setHh(int hh) {
        this.hh = hh;
    }
}
```
Its outputs is as follows:
```text
shanxi
85
3.1415
Test static block: 3.1415
2
23
```
# compile-time constant
Compile-time Constant: A variable use `final` modifier, its type is `primitive type` or `String`, and its value is a `constant expression`.
## What is constant expression?
[Constant Expressions in Oracle](https://docs.oracle.com/javase/specs/jls/se14/html/jls-15.html#jls-15.29)

e.g. Constant expressions:
```text
true
(short)(1*2*3*4*5*6)
Integer.MAX_VALUE / 2
2.0 * Math.PI
"The integer " + Long.MAX_VALUE + " is mighty big."
```

A constant expression is an expression denoting a value of primitive type or a String that does not complete abruptly and is composed using only the following:

- Literals of primitive type and literals of type String 

- Casts to primitive types and casts to type String

- The unary operators +, -, ~, and ! (but not `++` or `--`) 

- The multiplicative operators *, /, and % 

- The additive operators + and - 

- The shift operators <<, >>, and >>> 

- The relational operators <, <=, >, and >= (but not `instanceof`) 

- The equality operators == and != 

- The bitwise and logical operators &, ^, and | 

- The conditional-and operator && and the conditional-or operator || 

- The ternary conditional operator ? : 

- Parenthesized expressions whose contained expression is a constant expression.

- Simple names that refer to constant variables.

- Qualified names of the form TypeName . Identifier that refer to constant variables.

Constant expressions of type String are always "interned" so as to share unique instances, using the method String.intern.

A constant expression is always treated as FP-strict, even if it occurs in a context where a non-constant expression would not be considered to be FP-strict.

All in all, except for `instanceof`, `++`, `--`

# runtime constant
Runtime constant: A variable use `static` modifier, except for compile-time constant.

# Class file
Compile-time constants' reference will be replaced with the actual value, like as follows.
```java
package individual.cy.learn.mess;

class Test {
    public static final String NAME = "shanxi";
    public static final int SCORE = 85;
    public static final double PI = 3.1415D;
    public static final Integer CC = 2;
    public static int age = 23;
    private final int gg = 10;
    private int hh = 10;

    Test() {
    }

    public int getHh() {
        return this.hh;
    }

    public void setHh(int hh) {
        this.hh = hh;
    }

    static {
        System.out.println("Test static block: 3.1415");
    }
}
```
```java
package individual.cy.learn.mess;

public class CompileAndRuntimeConstant {
    public CompileAndRuntimeConstant() {
    }

    public static void main(String[] args) {
        System.out.println("shanxi");
        System.out.println(85);
        System.out.println(3.1415D);
        System.out.println(Test.CC);
        System.out.println(Test.age);
        System.out.println();
    }
}
```