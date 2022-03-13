---
title: What's thread in Java?
date: 2020-05-15T14:55:38+08:00
lastmod: 2020-05-15T15:25:01+08:00
slug: 5c028c36865949156511af9e4e298593
draft: false
categories: [java]
tags: [thread, java]
keywords: thread, multi-thread, os, java
description: Let's learn and create thread in Java.
---
# Lifecycle of a thread in Java

![Life cycle of a thread in Java](assets/thread-life-cycle.png)

![Lifecycle and States of a Thread in Java](assets/threadLifeCycle.jpg)

# How?

## Thread()

```java
public class CreateThreadByThreadClass extends Thread {
    @Override
    public void run() {
        int loopTimes = 100;
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```

## Runnable()

```java
public class CreateThreadByRunnableInterface implements Runnable {
    @Override
    public void run() {
        int loopTimes = 100;
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```

## Callable()

```java
public class CreateThreadByCallableInterface implements Callable<Integer> {
    @Override
    public Integer call() {
        int sum = 0, loopTimes = 100;
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            sum += i;
        }
        return sum;
    }
}
```

# Tester

```java
public class Tester {


    /**
     * 创建线程池
     */
    private static final ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("pool-%d").build();
    private static final ExecutorService executorService = new ThreadPoolExecutor(5, 200, 0L,
            TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(1024), threadFactory, new ThreadPoolExecutor.AbortPolicy());

    public static void main(String[] args) {
        int loopTimes = 100;
        //testThread(loopTimes);
        testRunnable(loopTimes);
        //testCallable(loopTimes);
    }

    private static void testRunnable(int loopTimes) {
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                CreateThreadByRunnableInterface myRunnable = new CreateThreadByRunnableInterface();
                // 不要显示创建线程,通过线程池创建
                executorService.execute(myRunnable);
                executorService.execute(myRunnable);
                executorService.shutdown();
            }
        }
    }

    private static void testThread(int loopTimes) {
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Thread thread1 = new CreateThreadByThreadClass();
                Thread thread2 = new CreateThreadByThreadClass();
                thread1.start();
                thread2.start();
            }
        }
    }

    private static void testCallable(int loopTimes) {
        for (int i = 0; i < loopTimes; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Callable<Integer> myCallable = new CreateThreadByCallableInterface();
                FutureTask<Integer> futureTask = new FutureTask<>(myCallable);
                executorService.submit(futureTask);
                executorService.shutdown();
            }
        }
    }
}

```


