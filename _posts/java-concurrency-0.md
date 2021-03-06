---
title: Java并发(1)-基础概念
date: 2016-06-22 16:30:26
categories: Java
tags: 
     - Java基础
     - 并发
     - 多线程
---


## 前言

这是将是一系列关于Java并发基础知识的文章。事实上，主要是《实战Java高并发程序设计》的读书笔记和网络资料以及对它们的整理。

## 相关概念

###  1.同步Synchronous和异步Asynchronous

 同步和异步通常是用来形容一次方法调用。
 
>所谓同步，就是在发出一个"调用"时，在没有得到结果之前，该"调用"就不返回。但是一旦调用返回，就得到返回值了。
换句话说，就是由"调用者"主动等待这个"调用"的结果。

>而异步则是相反，"调用"在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在"调用"发出后，"被调用者"通过状态、通知来通知调用者，或通过回调函数处理这个调用。

_来自>知乎[怎样理解阻塞非阻塞与同步异步的区别？](https://www.zhihu.com/question/19732473)_


### 2.并发、并行

>并发和并行的区别就是一个处理器同时处理多个任务和多个处理器或者是多核的处理器同时处理多个不同的任务。
>前者是逻辑上的同时发生，而后者是物理上的同时发生。
>
>并发性(concurrency)，又称共行性，是指能处理多个同时性活动的能力，并发事件之间不一定要同一时刻发生。

>并行(parallelism)是指同时发生的两个并发事件，具有并发的含义，而并发则不一定并行。
>
>来个比喻：并发和并行的区别就是一个人同时吃三个馒头和三个人同时吃三个馒头。


![并发与并行示例](/images/concurrency/concurrency_byb.jpg)

_来自> [并发和并行的区别：吃馒头的比喻](http://developer.51cto.com/art/200908/141553.htm)_


### 3. 临界区

用来表示一种公共资源可以被多个线程使用，但是每次只允许一个线程使用它，一旦临界区资源被占用，其他资源想要访问它只能等待。

### 4. 堵塞和非堵塞

堵塞和非堵塞用来形容线程相互影响的一种状态，比如一个线程占用了临界区资源，那么其他所以线程需要使用这个资源并且等待，等待会导致线程挂起，这便是堵塞。非堵塞相反，表示线程没有被其他线程妨碍，会不断尝试向前执行。

### 5. 死锁（Deadlock）、饥饿（Starvation）和活锁（Livelock）

死锁饥饿和活锁都是用来形容多线程的活跃性。

* 死锁：是指两个或者两个以上的进程执行过程中，由于竞争资源或者由于彼此通信而造成的一种堵塞的现象，若无外力的作用，他们都将无法推进下去。
* 饥饿：是指线程一直无法获取资源，导致一个堵塞。例如可能是因为优先级太低，而高优先级的线程一直抢占它需要的资源导致低级线程无法工作。
* 活锁：是指任务或者执行者没有被阻塞，由于某些条件没有满足，导致一直重复尝试，失败，尝试，失败。


## 并发级别

由于临界区的存在,多线程之间的并发必须受到控制。根据控制并发的策略,我们可以把并发的级别进行分类,大致上可以分为阻塞、无饥饿、无障碍、无锁、无等待几种。

### 1.堵塞（Blocking）

一个线程是阻塞的,那么在其他线程释放资源之前,当前线程无法继续执行。

### 2.无饥饿（Starvation-Free）

如果锁是公平的，满足所有的线程按照顺序排队（高优先级的线程不插队），那么所有的线程都有执行机会。

### 3.无障碍（Obstruction-Free）

表示所有线程都可以访问临界区，为了保证数据安全，如果线程发生冲突就回滚。

### 4.无锁（Lock-Free）

在无锁的情况下，所有的线程都能尝试对临界区进行访问，但不同的是，无锁的并发保证必然有一个线程能够在有限步内完成操作离开临界区。

### 5.无等待（Wait-Free）

无锁只要求有一个线程可以在有限步内完成操作,而无等待则在无锁的基础上更进一步进行扩展。它要求所有的线程都必须在有限步内完成,这样就不会引起饥饿问题。如果限制这个步骤上限,还可以进一步分解为有界无等待和线程数无关的无等待几种,它们之间的区别只是对循环次数的限制不同。

一种典型的无等待结构就是 RCU(Read-Copy-Update)。它的基本思想是,对数据的读可以不加控制。


## 原子性、可见性、有序性

Java内存模型是围绕着并发过程中如何处理原子性、可见性、有序性这三个特征来建立的，下面是这三个特性的实现原理：

### 1.原子性（Atomicity）

原子性表示一个操作不可中断，即使在多个线程一起执行的时候，一个操作一旦开始就不会被其他线程干扰。

### 2.可见性（Visibility）

可见性是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。

### 3.有序性（Ordering）

有序性问题的原因是因为程序在执行时，可能会进行指令重排，重排后的指令与原指令的顺序未必一致。



