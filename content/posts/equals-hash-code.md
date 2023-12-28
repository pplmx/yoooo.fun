---
categories:
    - java
date: 2020-05-13T14:32:32Z
description: Let's learn equals() and hashCode() in Java
keywords: hashCode, equals, 31
lastmod: 2020-05-13T14:53:35Z
tags:
    - hash
title: equals() and hashCode() in Java
---



# equals()

```java
// defined in Object class
public boolean equals(Object obj) {
	return (this == obj);
}
```

equals()的Override, 应该遵循以下原则:

- 自反性：**x.equals(x)** return `true`

- 对称性：**y.equals(x)** return `true` <=> **x.equals(y)** return `true`

- 传递性：**x.equals(y)** return `true`, **y.equals(z)** return `true` => **x.equals(z)** return `true`

- 一致性：无论对**x.equals(y)**调用多少次,结果都应该一致; 除非equals()中使用到的属性被更改了

- 非空性：任何非空的引用值X，x.equals(null)的返回值一定为false

# hashCode()

```java
// defined in Object class
@HotSpotIntrinsicCandidate
public native int hashCode();
```

- 在某个Java程序的`一次执行过程`中, 只要`equals()使用的属性`没有被改变, 那么`同一个对象`无论调用多少次hashCode(), 其返回的值都`必须一致`
- 在同一个Java程序的`多次执行过程`中, 一个对象的hashCode()返回值,`不必保持相同`

# Override Rule

> 为什么重写equals()的时候,必须重写hashCode()?
>
>   - Java的general contract规定, 相等的对象,必须具有相同的hash code
>   - Object对象默认实现的hashCode(), 是为每一个对象返回不同的值

equals()和hashCode()的Override, 应该遵循以下原则:

- 若`o1.equals(o2)`为true, 则`o1.hashCode() == o2.hashCode()`为true

- 反之, 不成立

- equals不相等的两个对象, hashCode可能相等

- However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

```text
o1.equals(o2)
==>
o1.hashCode() == o2.hashCode()
```

# Implementation

```java
public class User {
    private int id;
    private String name;
    private String email;

    @Override
    public int hashCode() {
        return Objects.hashCode(id, name, email);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof User user)) {
            return false;
        }
        return id == user.id &&
            Objects.equal(name, user.name) &&
            Objects.equal(email, user.email);
    }
}
```

## Another solution: hashCode()

```java
@Override
public int hashCode() {
    int result = id;
    result = 31 * result + name.hashCode();
    result = 31 * result + email.hashCode();
    return result;
}
```

## Objects.hashCode()

```java
public static int hashCode(@Nullable Object @Nullable ... objects) {
  return Arrays.hashCode(objects);
}
```

## Arrays.hashCode()

```java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

# Why is the multiplier 31?

- `选择质数`, 减少哈希碰撞(Hash Collision)
- 不选择`较小的质数`, e.g. 2; 因为冲突率依然很高
- 不选择`较大的质数`, e.g. 101; 会产生溢出
- `31 * i` 可被jvm优化为`(i << 5) - i`

