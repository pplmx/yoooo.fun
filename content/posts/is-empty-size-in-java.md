---
title: Time complexity in isEmpty() and size()
date: 2020-05-06T14:55:08+08:00
lastmod: 2020-05-06T15:05:32+08:00
slug: 76cb693819e47819d3fc64da34c9785a
draft: false
categories: [java]
tags: [complexity]
keywords: time complexity, complexity
description: Time complexity in isEmpty() and size()
---
# Time complexity in isEmpty() and size()

## Conclusion

isEmpty() is always O(1).

size() is mostly O(1), but it can be also O(n).

## Why?

Mostly, size() and isEmpty() implementation

```java
public int size() {
	return size;
}

public boolean isEmpty() {
	return size == 0;
}
```

For the above, both of their time complexity is O(1).

---

For **ConcurrentLinkedQueue** class

```java
/**
 * Returns the first live (non-deleted) node on list, or null if none.
 * This is yet another variant of poll/peek; here returning the
 * first node, not element.  We could make peek() a wrapper around
 * first(), but that would cost an extra volatile read of item,
 * and the need to add a retry loop to deal with the possibility
 * of losing a race to a concurrent poll().
 */
Node<E> first() {
    restartFromHead: for (;;) {
        for (Node<E> h = head, p = h, q;; p = q) {
            boolean hasItem = (p.item != null);
            if (hasItem || (q = p.next) == null) {
                updateHead(h, p);
                return hasItem ? p : null;
            }
            else if (p == q)
                continue restartFromHead;
        }
    }
}

/**
 * Returns {@code true} if this queue contains no elements.
 *
 * @return {@code true} if this queue contains no elements
 */
public boolean isEmpty() {
    return first() == null;
}

/**
 * Returns the number of elements in this queue.  If this queue
 * contains more than {@code Integer.MAX_VALUE} elements, returns
 * {@code Integer.MAX_VALUE}.
 *
 * <p>Beware that, unlike in most collections, this method is
 * <em>NOT</em> a constant-time operation. Because of the
 * asynchronous nature of these queues, determining the current
 * number of elements requires an O(n) traversal.
 * Additionally, if elements are added or removed during execution
 * of this method, the returned result may be inaccurate.  Thus,
 * this method is typically not very useful in concurrent
 * applications.
 *
 * @return the number of elements in this queue
 */
public int size() {
    restartFromHead: for (;;) {
        int count = 0;
        for (ConcurrentLinkedQueue.Node<E> p = first(); p != null;) {
            if (p.item != null)
                if (++count == Integer.MAX_VALUE)
                    break;  // @see Collection.size()
            if (p == (p = p.next))
                continue restartFromHead;
        }
        return count;
    }
}
```

For this, isEmpty() is O(1), size() is O(n).