---
title: Call to __init__ of super class is missed
date: 2020-09-03T09:06:03+08:00
slug: 84434c8d5d320a366d48e50658887a05
draft: false
lastmod: 2020-09-04T16:24:29+08:00
categories: [python]
tags: [general]
keywords: __init__, init, inherit, multi-inherit, python
description: You should invoke super().__init__() for every class.
---
# Call to `__init__` of super class is missed

## Origin

If a parent class declare `__init__` method explicitly, even if that method is empty

then the sub class's `__init__` method need to invoke parent's `__init__`

if not, a warning `Call to __init__ of super class is missed` will be thrown by PyCharm.

## Tracing

Why? Please follow me, look at the first demo

### Error Demo

```python
class Animal:
    def __init__(self):
        self.color = 'black'


class Cat(Animal):
    def __init__(self):
        self.age = 1


if __name__ == '__main__':
    ci = Cat()
    print(ci.color, ci.age)

# it will throw error: AttributeError: 'Cat' object has no attribute 'color'
```

### Fine 1

If a sub class do not override `__init__` method, parent's attributes will be inherited.

```python
class Animal:
    def __init__(self):
        self.color = 'black'


class Cat(Animal):
    pass


if __name__ == '__main__':
    ci = Cat()
    print(ci.color) # black

# this works fine, Animal's attr is be inherited.
```

### Fine 2

If a sub class want to declare its own attributes and inherit its parent's attributes, do as follows:

```python
class Animal:
    def __init__(self):
        self.color = 'black'


class Cat(Animal):
    def __init__(self):
        super().__init__()
        self.age = 1


if __name__ == '__main__':
    ci = Cat()
    print(ci.color, ci.age) # black 1
```

### Fine 3: Multiple Inheritance - solution 1

If a sub class inherits from multiple parent class, you should do like as follows.

If you do not **explicitly** invoke parent's `__init__` for **each** parent class in Subclass's `__init__`, it will **only** inherit **the first parent class**'s attributes.(The first parent class in the following code is **Engine** class.)

```python
class Engine:
    def __init__(self):
        self.performance = 80


class Skeleton:
    def __init__(self):
        self.shape = 'Rectangle'


class Car(Engine, Skeleton):
    def __init__(self):
        Engine.__init__(self)
        Skeleton.__init__(self)

    def deliver(self):
        print(self.performance)
        print(self.shape)


if __name__ == '__main__':
    BMW = Car()
    BMW.deliver()

```

### Fine 3: Multiple Inheritance - solution 2

Or like this:

(If you are not sure the parent class's behavior, Solution 1 is very good.)

```python
class Engine:
    def __init__(self):
        super().__init__()
        self.performance = 80


class Skeleton:
    def __init__(self):
        super().__init__()
        self.shape = 'Rectangle'


class Car(Engine, Skeleton):
    def __init__(self):
        super().__init__()

    def deliver(self):
        print(self.performance)
        print(self.shape)
```

## Conclusion

At the most time(Or **always**. Unless you know why you need not invoke init), you should invoke parent's init in every subclass init method.