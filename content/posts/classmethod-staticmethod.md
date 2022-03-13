---
title: Let's understand class and static method in Python for a Java-er
date: 2020-05-08T16:20:48+08:00
lastmod: 2020-05-08T16:34:11+08:00
slug: e034ce7185dbcbd648c3a4750ef8b566
draft: false
categories: [python]
tags: [python]
keywords: classmethod, staticmethod, java
description: How to understand class method and static method for a Java-er.
---
# @classmethod & @staticmethod

If you had the base knowledge in Java, maybe you would have been confused with it.

-   What is the class method?
-   What is the static method?
-   Are they the same as each other?

In Java, the class method does is the static method. They refer to the same thing.

Personally, 

-   @classmethod is the concept of static method(class method) in Java
-   @staticmethod, Without the corresponding concept in Java, it merely belongs to Python.

---

In Python, they are almost identical. Most of the use of @staticmethod can be replaced with @classmethod.

| @classmethod                       | @staticmethod                                    |
| ---------------------------------- | ------------------------------------------------ |
| <class 'method'> < bound method >  | <class 'function'> < function >                  |
| can be extended, can be overridden | can be extended, can be overridden               |
|                                    | can be changed with @classmethod when overridden |

# FYI

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from abc import abstractmethod


def fibonacci(idx):
    a, b = 1, 2
    while a <= idx + 1:
        yield a
        a, b = b, a + b


class Animal:

    @abstractmethod
    def run(self):
        raise NotImplementedError

    @classmethod
    def where_is(cls):
        print("Where is the animal?")

    @staticmethod
    def is_alive():
        return True


class Dog(Animal):

    def __init__(self, name, gender='male'):
        self.__name = name
        self.__gender = gender

    def run(self):
        print(f'Dog, {self.__name} runs fast.')

    # @classmethod
    # def where_is(cls):
    #     print("The dog is under the tree.")


class Cat(Animal):

    def __init__(self, name, gender='female'):
        self.__name = name
        self.__gender = gender

    def run(self):
        print(f'Cat, {self.__name} runs fast.')

    # @classmethod
    # def where_is(cls):
    #     print("The cat is on the tree.")


class Bird(Animal):

    def __init__(self, name, gender='female'):
        self.__name = name
        self.__gender = gender

    def run(self):
        print(f'Bird, {self.__name} flies fast.')

    @classmethod
    def where_is(cls):
        print("The bird is in the sky.")

    @staticmethod
    def is_alive():
        return False


class Horse(Animal):

    def __init__(self, name, gender='female'):
        self.__name = name
        self.__gender = gender

    def run(self):
        print(f'Horse, {self.__name} runs fast.')

    @classmethod
    def where_is(cls):
        print("The horse is on the grassland.")

    @classmethod
    def is_alive(cls):
        return False


if __name__ == '__main__':
    spotty = Dog("Spotty")
    mimi = Cat("Mimi")
    bee = Bird("Bee")
    hoo = Horse("Hoo")
    print(type(spotty.run), spotty.run)
    print(type(mimi.run), mimi.run)
    print(type(bee.run), bee.run)
    print(type(hoo.run), hoo.run)
    print()
    print(type(spotty.is_alive), spotty.is_alive)
    print(type(mimi.is_alive), mimi.is_alive)
    print(type(bee.is_alive), bee.is_alive)
    print(type(hoo.is_alive), hoo.is_alive)
    print()
    print(type(spotty.where_is), spotty.where_is)
    print(type(mimi.where_is), mimi.where_is)
    print(type(bee.where_is), bee.where_is)
    print(type(hoo.where_is), hoo.where_is)
    print()
    print(type(Dog.where_is), Dog.where_is)
    print(type(Cat.where_is), Cat.where_is)
    print(type(Bird.where_is), Bird.where_is)
    print(type(Horse.where_is), Horse.where_is)
    print()
    print(type(fibonacci), fibonacci)

```

```text
<class 'method'> <bound method Dog.run of <__main__.Dog object at 0x000001C8A0D99040>>
<class 'method'> <bound method Cat.run of <__main__.Cat object at 0x000001C8A0D990D0>>
<class 'method'> <bound method Bird.run of <__main__.Bird object at 0x000001C8A0D99130>>
<class 'method'> <bound method Horse.run of <__main__.Horse object at 0x000001C8A0D99190>>

<class 'function'> <function Animal.is_alive at 0x000001C8A0D8FCA0>
<class 'function'> <function Animal.is_alive at 0x000001C8A0D8FCA0>
<class 'function'> <function Bird.is_alive at 0x000001C8A0DA2160>
<class 'method'> <bound method Horse.is_alive of <class '__main__.Horse'>>

<class 'method'> <bound method Animal.where_is of <class '__main__.Dog'>>
<class 'method'> <bound method Animal.where_is of <class '__main__.Cat'>>
<class 'method'> <bound method Bird.where_is of <class '__main__.Bird'>>
<class 'method'> <bound method Horse.where_is of <class '__main__.Horse'>>

<class 'method'> <bound method Animal.where_is of <class '__main__.Dog'>>
<class 'method'> <bound method Animal.where_is of <class '__main__.Cat'>>
<class 'method'> <bound method Bird.where_is of <class '__main__.Bird'>>
<class 'method'> <bound method Horse.where_is of <class '__main__.Horse'>>

<class 'function'> <function fibonacci at 0x000001C8A0D8FA60>

Process finished with exit code 0

```
