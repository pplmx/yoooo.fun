---
title: How to use switch in Python?
date: 2020-06-23T10:43:33+08:00
slug: 34cadb1339b7a97f1890c90610c908f4
draft: false
lastmod: 2020-06-23T10:45:43+08:00
categories: [python]
tags: [switch]
keywords: switch, switch case in python
description: Let's use switch in Python.
---
# To implement a switch structure in Python

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

switch = {
    'add': lambda x, y: x + y,
    'sub': lambda x, y: x - y,
    'mul': lambda x, y: x * y,
    'div': lambda x, y: x / y,
}

if __name__ == '__main__':
    print(switch['add'](1, 8))
    print(switch['sub'](1, 8))
    print(switch['mul'](1, 8))
    print(switch['div'](1, 8))

```

```text
9
-7
8
0.125
```