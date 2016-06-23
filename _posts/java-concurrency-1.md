---
title: Java并发(2)-并发基础
date: 2016-06-23 19:30:26
categories: Java
tags: 
     - Java基础
     - 并发
     - 多线程
---


## 进程与线程

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配的基本单位，是操作系统结构的基础。在早期面向进程设计的计算机结构中，进程是程序的基本执行实体；在当代面向线程设计的计算机结构中，进程是线程的容器。程序是指令、数据及其组织形式的描述，进程是程序的实体。

线程(Thread)是操作系统能够进行运算调度的最小单位。它被包涵在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

>进程让操作系统的并发性成为可能，而线程让进程的内部并发成为可能。
>一个进程虽然包括多个线程，但是这些线程是共同享有进程占有的资源和地址空间的。进程是操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位。

来自> _[Java并发编程：进程和线程之由来](http://www.cnblogs.com/dolphin0520/p/3910667.html)_

## 线程的生命周期（线程的状态）

```
public enum State {
   NEW,// 刚刚创建的线程，还没开始执行的时候
   RUNNABLE, // 执行的时候
   BLOCKED, // 等待锁的的时候，进入堵塞状态
   WAITING, // 等待，无时间限制，wait通过notify唤醒，join通过目标线程终止后唤醒
   TIMED_WAITING, // 进行一个 有时限的等待
   TERMINATED; // 线程执行完毕，进入这个状态，表示结束。
}
```

粗略的画了一张图：

![线程的状态](/images/concurrency/concurrency_thread_state.png)


## 初始线程：线程的基本操作

### 1.新建线程

新建线程很简单。只要使用 new 关键字创建一个线程对象,并且将它start()起来即可。

**如果只调用run方法，那么只会在当前线程中执行run方法，只是作为一个普通的方法调用。**

线程的创建有两种方式：

```
Thread t1=new Thread(){
   @Override
   public void run(){
       System.out.println("Hello, I am t1");
} };
t1.start();
```

```
Thread t1=new Thread(new MyRunnable());
t1.start();
```

### 2.线程的终止

stop()方法已经废弃，因为stop()方法太过于暴力,强行把执行到一半的线程终止,可能会引起一些数据不一致的问题。建议的方式是定义了一个标记变量,用于指示线程是否需要退出。

```
public class StopThreadUnsafe {
    public static User u = new User();

    public static class User {
        private int id;
        private String name;

        public User() {
            id = 0;
            name = "0";
        }

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "User [id=" + id + ", name=" + name + "]";
        }
    }

    public static class ChangeObjectThread extends Thread {

        @Override
        public void run() {
            while (true) {
                synchronized (u) {
                    int v = (int) (System.currentTimeMillis() / 1000);
                    u.setId(v);
                    //Oh, do sth. else
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    u.setName(String.valueOf(v));
                }
                Thread.yield();
            }
        }
    }

    public static class ReadObjectThread extends Thread {
        @Override
        public void run() {
            while (true) {
                synchronized (u) {
                    if (u.getId() != Integer.parseInt(u.getName())) {
                        System.out.println(u.toString());
                    }
                }
                Thread.yield();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {

        new ReadObjectThread().start();

        while (true) {
            ChangeObjectThread t = new ChangeObjectThread();
            t.start();
            Thread.sleep(150);
            t.stopMe();
        }
    }
}
```
结果：

```
User [id=1466487989, name=1466487988]
User [id=1466487990, name=1466487989]
User [id=1466487990, name=1466487989]
User [id=1466487990, name=1466487989]
User [id=1466487990, name=1466487989]
```

添加标记

```
public static class ChangeObjectThread extends Thread {

   volatile boolean stopMe = false;

   public void stopMe() {
       stopMe = true;
   }

   @Override
   public void run() {
       while (true) {
           synchronized (u) {
               if (stopMe) {
                   System.out.println("exit by stop me");
                   break;
               }
               int v = (int) (System.currentTimeMillis() / 1000);
               u.setId(v);
               //Oh, do sth. else
               try {
                   Thread.sleep(100);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               u.setName(String.valueOf(v));
           }
           Thread.yield();
       }
   }
}
```

使用stopMe结束线程。

### 3.线程中断

>在 Java 中，线程中断是一种重要的线程协作机制。从表面上理解，中断就是让目标线程停止执行的意思，实际上并非完全如此。严格地讲，线程中断并不会使线程立即退出，而是给线程发送一个通知，告知目标线程， 有人希望你退出啦！至于目标线程接到通知后如何处理，则完全由目标线程自行决定。

相关的方法：

```
public void Thread.interrupt() // 中断线程
public boolean Thread.isInterrupted() // 判断是否被中断
public static boolean Thread.interrupted() // 判断是否被中断,并清除当前中断状态
```

```
public class InterruptedTest {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                while (true) {
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println("Interruted!");
                        break;
                    }
                    System.out.println("=========running=========");
                    try {
                        Thread.sleep(600);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        System.out.println("Interrupted When Sleep");
                        Thread.currentThread().interrupt();
                    }
                    System.out.println("==running after sleep==");
                    Thread.yield();
                }
            }
        };
        t1.start();
        Thread.sleep(2000);
        t1.interrupt();
    }
}
```

* 如果不使用Thread.currentThread().isInterrupted()判断退出的话，是不会退出的。
* 如果添加Thread.sleep，那么要注意这个方法会由于线程的中断发生异常，并且此时会清除中断标记。所以在异常处理中,需要再次设置中断标记位。


### 等待（wait）和通知（notify）

为了支持多线程的协作，JDK提供了两个非常重要的方法wait和notify。这两个方法是属于Object类的方法。他们的签名是：

```
public final void wait() throws InterruptedException
public final native void notify()
```

>当在一个对象实例上调用 wait()方法后,当前线程就会在这个对象上等待。这是什么意思呢?比如,线程 A 中，调用了obj.wait()方法，那么线程 A 就会停止继续执行，而转为等待状态。 等待到何时结束呢？线程 A 会一直等到其他线程调用了 obj.notify()方法为止。这时，obj 对象就俨然成为多个线程之间的有效通信手段。


```
public class WaitAndNotifyDemo {

    static final Object obj = new Object();

    static class T1 extends Thread {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println("===T1 begin===");
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("===T1 end===");
            }
        }
    }

    static class T2 extends Thread {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println("===T2 begin===");
                obj.notifyAll();
//                obj.notify();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("===T2 end===");
            }
        }
    }

    static class T3 extends Thread {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println("===T3 begin===");
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("===T3 end===");
            }
        }
    }

    public static void main(String[] args) {
        new T1().start();
        new T3().start();
        new T2().start();
    }
}
```

```
===T1 begin===
===T3 begin===
===T2 begin===
===T2 end===
===T3 end===
===T1 end===
```


如例，wait和notify是对锁对象进行操作的，并且必须在 synchronized 函数或者代码块里面。wait的时候会释放锁，然后线程进入等待（WAITING），等待其他线程调用notify或者notifyAll方法（BLOCKED）并释放锁，然后wait线程获得锁继续执行（RUNNABLE）。假如在notify调用的时候锁还没被释放，那么还需要继续等待，直到重新获得锁。

> **注意：**当notify()方法会从这个等待队列中，随机选择一个线程，并将其唤醒。而notifyAll()方法会唤醒在这个等待队列中所有等待的线程，而不是随机选择一个。

**还要注意wait和sleep**   wait和sleep都可以是线程进入等待。但是要注意他们之间的区别：

1. 这两个方法来自不同的类分别是Thread和Object。
2. 最主要是sleep方法没有释放锁，而wait方法释放了锁。
3. wait，notify和notifyAll只能在synchronized同步控制方法或者synchronized同步控制块里面使用，而sleep没有这个限制。
4. sleep可以指定等待的时间，并且线程状态变为TIMED_WAITING。wait的等待时间不确定（当然还有一个wait超时方法，具有一个最长等待时间，状态也是变成TIMED_WAITING），需要根据notify的调用和锁的释放，并且线程进入的状态为WAITING。



### 5.挂起（suspend）和继续执行（resume）线程

>不推荐使用 suspend()去挂起线程的原因,是因为 suspend()在导致线程暂停的同时,并不会 去释放任何锁资源。此时,其他任何线程想要访问被它暂用的锁时,都会被牵连,导致无法正 常继续运行(如图 2.7 所示)。直到对应的线程上进行了 resume()操作,被挂起的线程才能继续, 从而其他所有阻塞在相关锁上的线程也可以继续执行。但是,如果 resume()操作意外地在 suspend()前就执行了,那么被挂起的线程可能很难有机会被继续执行。并且,更严重的是:它 所占用的锁不会被释放,因此可能会导致整个系统工作不正常。而且,对于被挂起的线程,从 它的线程状态上看,居然还是 Runnable,这也会严重影响我们对系统当前状态的判断。

那么如何完成线程的挂起和继续呢？使用wait和notify。

```
public class GoodSuspend {
    public static Object u = new Object();

    public static class ChangeObjectThread extends Thread {
        volatile boolean suspendme = false;

        public void suspendMe() {
            suspendme = true;
        }

        public void resumeMe() {
            suspendme = false;
            synchronized (this) {
                notify();
            }
        }

        @Override
        public void run() {
            while (true) {

                if (Thread.currentThread().isInterrupted()){
                    System.out.println("exit change");
                    break;
                }

                synchronized (this) {
                    while (suspendme) {
                        try {
                            wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            interrupt();
                        }
                    }
                }
                synchronized (u) {
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        interrupt();
                    }
                    System.out.println("in ChangeObjectThread");
                }
                Thread.yield();
            }
        }
    }

    public static class ReadObjectThread extends Thread {
        @Override
        public void run() {
            while (true) {

                if (Thread.currentThread().isInterrupted()){
                    System.out.println("exit read");
                    break;
                }

                synchronized (u) {
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        interrupt();
                    }
                    System.out.println("in ReadObjectThread");
                }
                Thread.yield();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ChangeObjectThread t1 = new ChangeObjectThread();
        ReadObjectThread t2 = new ReadObjectThread();
        t1.start();
        t2.start();
        Thread.sleep(1000);
        t1.suspendMe();
        System.out.println("suspend t1 2 sec");
        Thread.sleep(2000);
        System.out.println("resume t1");
        t1.resumeMe();

        Thread.sleep(2000);
        t1.interrupt();
        t2.interrupt();

    }
}

```
1. 前1秒，两个线程同时执行；
2. 之后2秒t1挂起，只有t2执行；
3. 然后t1、t2同时执行2秒
4. 最后t1、t2中断

```
in ChangeObjectThread
in ChangeObjectThread
in ReadObjectThread
in ReadObjectThread
in ChangeObjectThread
in ChangeObjectThread
in ReadObjectThread
in ReadObjectThread
in ChangeObjectThread
suspend t1 2 sec
in ReadObjectThread
in ChangeObjectThread
in ReadObjectThread
in ReadObjectThread
in ReadObjectThread
...
in ReadObjectThread
in ReadObjectThread
in ReadObjectThread
resume t1
in ReadObjectThread
in ChangeObjectThread
in ReadObjectThread
in ChangeObjectThread
in ReadObjectThread
in ChangeObjectThread
in ReadObjectThread
in ChangeObjectThread
in ReadObjectThread
in ReadObjectThread
in ChangeObjectThread
exit change
in ReadObjectThread
exit read
```

### 6.等待线程结束（join）和谦让（yield）

```java
public final void join() throws InterruptedException
public final synchronized void join(long millis) throws InterruptedException
```

第一个表示无限等待，会一直堵塞当前线程。第二个给出了个最大等待时间，超过时间继续执行。

```java
public class JoinDemo {
    static int a;

    static class AddThread extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                ++a;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AddThread addThread = new AddThread();
        addThread.start();
        addThread.join();
        System.out.println(a);
    }
}
```

```java
public class JoinDemo {
    static int a;

    static class AddThread extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                ++a;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AddThread addThread = new AddThread();
        addThread.start();
        addThread.join(500);
        System.out.println(a);
    }
}
```

> 同时对于join要注意，join()的本质是让调用线程wait()在当前线程对象实例上，因此不要在应用程序中，在Thread对象实例上使用类似wait()或者notify()等方法,因为这很有可能会影响系统API的工作，或者被系统API所影响。

Thread.yield()方法表示让出CPU，但是它还回去竞争CPU资源。



## volatile


参考[这里](http://www.cnblogs.com/dolphin0520/p/3920373.html)和[这里](http://www.cnblogs.com/aigongsi/archive/2012/04/01/2429166.html)

>一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

>1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

>2）禁止进行指令重排序。


volatile关键字作用：

1. 保证可见性
2. 不一定保证原子性
3. 一定程度上的保证有序性



## 线程组

>在一个系统中，如果线程数量很多，而且功能分配比较明确，就可以将相同功能的线程放置在一个线程组里。打个比方，如果你有一个苹果，你就可以把它拿在手里，但是如果你有十 个苹果，你就最好还有一个篮子，否则不方便携带。对于多线程来说，也是这个道理。想要轻 处理几十个甚至上百个线程，最好还是将它们都装进对应的篮子里。

```
public class ThreadGroupName implements Runnable {


    @Override
    public void run() {
        String name = Thread.currentThread().getThreadGroup().getName() + "-" + Thread.currentThread().getName();
        while (true) {
            if (Thread.currentThread().isInterrupted()) break;
            System.out.println("I am " + name);
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
                Thread.currentThread().interrupt();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup tg = new ThreadGroup("PrintGroup");
        Thread t1 = new Thread(tg, new ThreadGroupName(), "T1");
        Thread t2 = new Thread(tg, new ThreadGroupName(), "T2");

        t1.start();
        t2.start();

        System.out.println(tg.activeCount());
        tg.list();

        Thread.sleep(4000);
        tg.interrupt();

    }
}
```

通过组合的形式，方便统一操作线程。



