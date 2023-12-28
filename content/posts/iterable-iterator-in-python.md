---
categories:
    - python
date: 2020-05-06T16:16:51Z
description: What's Iterable, Iterator, Generator in Python?
keywords: iterable, iterator, generator
lastmod: 2020-05-06T16:39:53Z
tags:
    - source
title: Difference among Iterable, Iterator and Generator
---



# Iterable(metaclass=ABCMeta)

```python
"""
__iter__
"""

@abstractmethod
def __iter__(self):
    while False:
        yield None

@classmethod
def __subclasshook__(cls, C):
    if cls is Iterable:
        return _check_methods(C, "__iter__")
    return NotImplemented
```

# Iterator(Iterable)

```python
"""
__next__
"""

@abstractmethod
def __next__(self):
    'Return the next item from the iterator. When exhausted, raise StopIteration'
    raise StopIteration

def __iter__(self):
    return self

@classmethod
def __subclasshook__(cls, C):
    if cls is Iterator:
        return _check_methods(C, '__iter__', '__next__')
    return NotImplemented
```

# Generator(Iterator)

```python
"""
send()
throw()
close()
"""

def __next__(self):
    """Return the next item from the generator.
    When exhausted, raise StopIteration.
    """
    return self.send(None)

@abstractmethod
def send(self, value):
    """Send a value into the generator.
    Return next yielded value or raise StopIteration.
    """
    raise StopIteration

@abstractmethod
def throw(self, typ, val=None, tb=None):
    """Raise an exception in the generator.
    Return next yielded value or raise StopIteration.
    """
    if val is None:
        if tb is None:
            raise typ
        val = typ()
    if tb is not None:
        val = val.with_traceback(tb)
    raise val

def close(self):
    """Raise GeneratorExit inside generator.
    """
    try:
        self.throw(GeneratorExit)
    except (GeneratorExit, StopIteration):
        pass
    else:
        raise RuntimeError("generator ignored GeneratorExit")

@classmethod
def __subclasshook__(cls, C):
    if cls is Generator:
        return _check_methods(C, '__iter__', '__next__',
                              'send', 'throw', 'close')
    return NotImplemented
```

# Conclusion

**Generator** is **Iterator**.

**Iterator** is **Iterable**.

---

According to `__subclasshook__` function,

Iterable, must implement `__iter__`

Iterator, must implement `__iter__` and `__next__`

Generator, must implement `__iter__` , `__next__`, `send`, `throw` and `close`

