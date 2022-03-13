---
title: __str__ and __repr__ in Python
date: 2020-06-07T22:05:53+08:00
lastmod: 2020-06-07T22:47:00+08:00
slug: b846a9692479a22f110ff7652a517dd2
draft: false
categories: [python]
tags: [general]
keywords: __str__, __repr__, python
description: What's difference between __str__ and __repr__ ?
---
# `__str__` and `__repr__` 

## `__str__`

- `print(obj)` 会调用`__str__`
- 如果`__str__`没有被重写, 那么将调用`__repr__`

## `__repr__`

- 在交互式环境下, 直接输入对象, 打印会调用`__repr__`

## Some Analysis

### Condition 1

- 只重写`__str__`
- `__repr__`调用父类

```python
class Apple(object):

    def __repr__(self) -> str:
        return super().__repr__()

    def __str__(self) -> str:
        return f'This is a {self.__color} apple.[Print by __str__]'

    def __init__(self, color):
        self.__color = color
```

---

```python
λ python
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> from base_knowledge import Apple
>>> red_apple = Apple("red")
>>> red_apple
<base_knowledge.Apple object at 0x000002BCE2110AF0>
>>> print(red_apple)
This is a red apple.[Print by __str__]
>>>
```

从以上输出, 

- print调用了`__str__`
- 交互式环境,直接输出了地址信息

### Condition 2

- 只重写`__repr__`
- `__str__`调用父类

```python
class Apple(object):

    def __repr__(self) -> str:
        return f'This is a {self.__color} apple.[Print by __repr__]'

    def __str__(self) -> str:
        return super().__str__()

    def __init__(self, color):
        self.__color = color
```

---

```python
λ python
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> from base_knowledge import Apple
>>> red_apple = Apple("red")
>>> red_apple
This is a red apple.[Print by __repr__]
>>> print(red_apple)
This is a red apple.[Print by __repr__]
>>>
```

从以上输出, 

- 交互式环境, 调用了重写的`__repr__`
- print也调用了`__repr__`

### Condition 3

- 重写`__str__`
- 重写`__repr__`

```python
class Apple(object):

    def __repr__(self) -> str:
        return f'This is a {self.__color} apple.[Print by __repr__]'

    def __str__(self) -> str:
        return f'This is a {self.__color} apple.[Print by __str__]'

    def __init__(self, color):
        self.__color = color
```

---

```python
λ python
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> from base_knowledge import Apple
>>> red_apple = Apple("red")
>>> red_apple
This is a red apple.[Print by __repr__]
>>> print(red_apple)
This is a red apple.[Print by __str__]
>>>
```

从以上输出, 

- 交互式环境, 直接输出, 调用了`__repr__`
- print, 调用了`__str__`

### Condition 4

- 都没重写

```python
class Apple(object):

    def __init__(self, color):
        self.__color = color
```

---

```python
λ python
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> from base_knowledge import Apple
>>> red_apple = Apple("red")
>>> red_apple
<base_knowledge.Apple object at 0x0000025ABFE10AF0>
>>> print(red_apple)
<base_knowledge.Apple object at 0x0000025ABFE10AF0>
>>>
```



结合Condition1, 2, 3, 4, 可以得出以下结论

- `__str__`如果没有被重写, 默认调用`__repr__`
- 交互式环境下, 直接输出, 一定调用`__repr__`(重写, 就调用重写的方法; 未重写, 就调用父类的)