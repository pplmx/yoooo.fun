---
title: Difference between OOP and POP
date: 2020-03-22T21:10:26+08:00
lastmod: 2020-03-22T21:28:11+08:00
slug: aa1a40ba6005de4e334c1c527ff8efae
draft: false
categories: [general]
tags: [OOP,POP]
keywords: OOP, POP
description: Difference between POP and OOP
---
# General Information

## Procedure Oriented Programming(POP)

>    [Procedural Programming](https://www.geeksforgeeks.org/introduction-of-programming-paradigms/) can be defined as a programming model which is derived from structured programming, based upon the concept of calling procedure. Procedures, also known as routines, subroutines or functions, simply consist of a series of computational steps to be carried out. During a program’s execution, any given procedure might be called at any point, including by other procedures or itself. 

## Object Oriented Programming(OOP)

>    [Object oriented programming](https://www.geeksforgeeks.org/basic-concepts-of-object-oriented-programming-using-c/) can be defined as a programming model which is based upon the concept of objects. Objects contain data in the form of attributes and code in the form of methods. In object oriented programming, computer programs are designed using the concept of objects that interact with real world. Object oriented programming languages are various but the most popular ones are class-based, meaning that objects are instances of classes, which also determine their types. 

## Difference between POP and OOP

|                             POP                              |                             OOP                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|  Program is divided into small parts called **functions**.   |   Program is divided into small parts called **objects**.    |
|              POP follows **top-down approach**.              |             OOP follows **bottom-up approach**.              |
|   There is no access specifier in procedural programming.    | Object oriented programming have access specifiers like private, public, protected etc. |
|          Adding new data and function is not easy.           |            Adding new data and function is easy.             |
| POP does not have any proper way for hiding data so it is **less secure**. |      OOP provides data hiding so it is **more secure**.      |
|                  Overloading is impossible.                  |                   Overloading is possible.                   |
|            Function is more important than data.             |            Data is more important than function.             |
|              POP is based on **unreal world**.               |               OOP is based on **real world**.                |
|         FORTRAN, ALGOL, COBOL,  BASIC, Pascal and C.         | Java, C++, C#, Python,  PHP, JavaScript, Ruby, Perl,  Objective-C, Dart, Swift, Scala. |

## Interpreted Languages

>   Interpreters will run through a program line by line and execute each command. Now, if the author decided he wanted to use a different kind of olive oil, he could scratch the old one out and add the new one. Your translator friend can then convey that change to you as it happens.
>
>   Interpreted languages were once known to be significantly slower than compiled languages. But, with the development of [just-in-time compilation](https://guide.freecodecamp.org/computer-science/just-in-time-compilation), that gap is shrinking.
>
>   Examples of common interpreted languages are PHP, Ruby, Python, and JavaScript.

## Compiled Languages

>   Compiled languages are converted directly into machine code that the processor can execute. As a result, they tend to be faster and more efficient to execute than interpreted languages. They also give the developer more control over hardware aspects, like memory management and CPU usage.
>
>   Compiled languages need a “build” step - they need to be manually compiled first. You need to “rebuild” the program every time you need to make a change. In our hummus example, the entire translation is written before it gets to you. If the original author decided he wanted to use a different kind of olive oil, the entire recipe would need to be translated again and then sent to you.
>
>   Examples of pure compiled languages are C, C++, Erlang, Haskell, Rust, and Go.

#### Advantages of Compiled Languages

Programs compiled into native code at compile time usually tend to be faster than those translated at run time, due to the overhead of the translation process.

#### Disadvantages of Compiled Languages

The most notable disadvantages are :-

-   Additional time needed to complete the entire compilation step before testing, and
-   Platform dependence of the generated binary code.

#### Advantages of Interpreted Languages

An Interpreted language gives implementations some additional flexibility over compiled implementations. Because interpreters execute the source program code themselves, the code itself is platform independent (Java’s byte code, for example). Other features include dynamic typing, and smaller executable program size.

#### Disadvantages of Interpreted Languages

The most notable disadvantage is typical execution speed compared to compiled languages.
