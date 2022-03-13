---
title: Override&Overload in Java and Python
date: 2020-04-25T21:56:25+08:00
slug: 0b982bfffbc69e5e89e25f1b238187e3
draft: false
lastmod: 2020-04-25T22:28:18+08:00
categories: [general]
tags: [java, python]
keywords: override, overload, difference, java, python
description: What's difference between Override and Overload? In java, or Python.
---
# Override & Overload in Java & Python

## In Java

| Overload                       | Override                           |
| ------------------------------ | ---------------------------------- |
| 参数列表: 必须**不同**         | 参数列表: 必须**一致**             |
| 返回类型: 可以相同, 也可以不同 | 返回类型: 相同, 或为派生类型       |
| 一种**编译时多态**例子         | 一种**运行时多态**的例子           |
| 重载发生在**同一个类**         | 重写发生在两个关系为**is-A**的类中 |

## In Python

| Overload                           | Override                               |
| ---------------------------------- | -------------------------------------- |
| 没有重载(以下列出原因)             | 基本与java一致                         |
| 重载要素: 1. 参数类型; 2. 参数数量 | (但是以下代码可运行, 只是不建议这样写) |
| Python可以接受任意类型的参数       |                                        |
| Python可以使用缺省参数             |                                        |
|                                    |                                        |

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class f:
    def add(self, a, b):
        return a + b


class e(f):
    def add(self, a, b, d):
        return a + b + d


if __name__ == '__main__':
    a = e()
    print(a.add(3, 5, 7))
```