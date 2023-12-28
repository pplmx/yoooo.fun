---
categories:
    - python
date: 2023-09-03T21:57:12Z
description: How to release a user-defined resource automatically?
keywords: python, context lib, context manager, with, resource
tags:
    - python
    - with
    - resource
    - context manager
title: Auto release custom func resource in Python
---



# how to release a user-defined resource automatically

## Problem

You want to release a resource returned by a custom function automatically when the resource is no longer needed.

## Solution

According to the [Python documentation](https://docs.python.org/3/reference/datamodel.html#context-managers), a context manager is an object that
defines the runtime context to be established when executing a `with` statement. The context manager's `__enter__()` method is called when the `with`
statement is entered and the context manager's `__exit__()` method is called when the `with` statement is exited. The `__exit__()` method is passed
the exception type, value, and traceback. If the `with` statement exits normally, the exception type, value, and traceback are `None`.

Use the `contextlib.contextmanager` decorator to create a context manager. The function must return a generator that yields exactly one value. The
value yielded is bound to the variable in the `with` statement's `as` clause. The generator's code block is executed when the `with` statement is
entered and exited.

The following example shows how to use the `contextlib.contextmanager` decorator to create a context manager from a function that returns a resource
object.

```python
import contextlib

@contextlib.contextmanager
def managed_custom_resource(*args, **kwargs):
    resource = acquire_resource(*args, **kwargs)
    try:
        yield resource
    finally:
        release_resource(resource)

def acquire_resource(*args, **kwargs):
    print('acquire_resource(*args={}, **kwargs={})'.format(args, kwargs))

    # Code to acquire resource, e.g.:
    resource = object()
    return resource

def release_resource(resource):
    print('release_resource(resource={})'.format(resource))

    # Code to release resource, e.g.:
    pass
```
