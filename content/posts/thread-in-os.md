---
title: Thread in OS
date: 2020-05-15T14:55:32+08:00
slug: 4dd9c49996f8b1dc7c8127ad0820ddab
draft: false
lastmod: 2020-05-15T15:25:01+08:00
categories: [general]
tags: [thread,os]
keywords: thread, multi-thread, os
description: Let's learn What's thread.
---
# Overview

## What's thread?

>   A thread is a path of execution within a process. A process can contain multiple threads.
>
>   **Thread** is an execution unit which consists of its own program counter, a stack, and a set of registers. Threads are also known as Lightweight processes. Threads are popular way to improve application through parallelism. The CPU switches rapidly back and forth among the threads giving illusion that the threads are running in parallel.
>
>   As each thread has its own independent resource for process execution, multpile processes can be executed parallely by increasing number of threads.
>
>   A thread is a flow of execution through the process code, with its own program counter that keeps track of which instruction to execute next, system registers which hold its current working variables, and a stack which contains the execution history.
>
>   A thread shares with its peer threads few information like code segment, data segment and open files. When one thread alters a code segment memory item, all other threads see that.
>
>   A thread is also called a **lightweight process**. Threads provide a way to improve application performance through parallelism. Threads represent a software approach to improving performance of operating system by reducing the overhead thread is equivalent to a classical process.
>
>   Each thread belongs to exactly one process and no thread can exist outside a process. Each thread represents a separate flow of control. Threads have been successfully used in implementing network servers and web server. They also provide a suitable foundation for parallel execution of applications on shared memory multiprocessors. The following figure shows the working of a single-threaded and a multithreaded process.

![Single Threaded and Multithreaded Process](/assets/single-and-multithreaded-process.png)

## Why multithreading?

>   A thread is also known as lightweight process. The idea is to achieve parallelism by dividing a process into multiple threads. For example, in a browser, multiple tabs can be different threads. MS Word uses multiple threads: one thread to format the text, another thread to process inputs, etc.

## Difference between Process and Thread

| S.N. | Process                                                      | Thread                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | Process is heavy weight or resource intensive.               | Thread is light weight, taking lesser resources than a process. |
| 2    | Process switching needs interaction with operating system.   | Thread switching does not need to interact with operating system. |
| 3    | In multiple processing environments, each process executes the same code but has its own memory and file resources. | All threads can share same set of open files, child processes. |
| 4    | If one process is blocked, then no other process can execute until the first process is unblocked. | While one thread is blocked and waiting, a second thread in the same task can run. |
| 5    | Multiple processes without using threads use more resources. | Multiple threaded processes use fewer resources.             |
| 6    | In multiple processes each process operates independently of the others. | One thread can read, write or change another thread's data.  |

## Pros of Thread over Process

-   *Responsiveness*

    >   If the process is divided into multiple threads, if one thread completes its execution, then its output can be immediately returned.

-   *Faster context switch*

    >   Context switch time between threads is lower compared to process context switch. Process context switching requires more overhead from the CPU.

-   *Effective utilization of multiprocessor system*

    >   If we have multiple threads in a single process, then we can schedule multiple threads on multiple processor. This will make process execution faster.

-   *Resource sharing*

    >   Resources like code, data, and files can be shared among all threads within a process.

-   *Communication*

    >   Communication between multiple threads is easier, as the threads shares common address space. while in process we have to follow some specific communication technique for communication between two process.

-   *Enhanced throughput of the system*

    >   If a process is divided into multiple threads, and each thread function is considered as one job, then the number of jobs completed per unit of time is increased, thus increasing the throughput of the system.

# Thread State

-   Creation

-   Ready

-   Running

-   Blocked

-   Dead

