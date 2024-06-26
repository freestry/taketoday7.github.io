---
layout: post
title:  线程转储中的“Locked Ownable Synchronizers”是什么？
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将了解线程锁定的可拥有同步器的含义。我们将编写一个使用Lock进行同步的简单程序，并在[线程转储](https://www.baeldung.com/java-thread-dump)中查看它的样子。

## 2. 什么是锁定的可拥有同步器？

每个线程都可能有一个同步器对象列表。**该列表中的条目表示线程已为其获取锁的可拥有同步器**。

AbstractOwnableSynchronizer类的实例可以用作同步器。它最常见的子类之一是Sync类，它是Lock接口实现的一个字段，如ReentrantReadWriteLock。

当我们调用ReentrantReadWriteLock.lock()方法时，代码在内部将其委托给Sync.lock()方法。**一旦我们获得了锁，Lock对象就会被添加到线程的锁定的可拥有同步器列表中**。

我们可以在典型的线程转储中查看此列表：

```bash
"Thread-0" #1 prio=5 os_prio=0 tid=0x000002411a452800 nid=0x1c18 waiting on condition [0x00000051a2bff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at cn.tuyucheng.taketoday.ownablesynchronizers.Application.main(Application.java:25)

   Locked ownable synchronizers:
        - <0x000000076e185e68> (a java.util.concurrent.locks.ReentrantReadWriteLock$FairSync)
```

根据我们用来生成它的工具，我们可能必须提供一个特定的选项。例如，我们使用[jstack](https://www.baeldung.com/java-thread-dump#1-jstack)运行以下命令：

```bash
jstack -l <pid>
```

使用-l选项，我们告诉jstack打印有关锁的其他信息。

## 3. 锁定的可拥有同步器如何提供帮助

**可拥有的同步器列表可帮助我们识别可能的应用程序死锁**。例如，我们可以在线程转储中看到名为Thread-1的不同线程是否正在等待获取同一Lock对象上的锁：

```bash
"Thread-1" #12 prio=5 os_prio=0 tid=0x00000241374d7000 nid=0x4da4 waiting on condition [0x00000051a42fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e185e68> (a java.util.concurrent.locks.ReentrantReadWriteLock$FairSync)
```

线程Thread-1处于WAITING状态。具体来说，它等待获取ID为<0x000000076e185e68\>的对象上的锁。但是，同一对象在线程Thread-0的锁定的可拥有同步器列表中。**我们现在知道，在线程Thread-0释放自己的锁之前，线程Thread-1无法继续**。

如果同时相同的情况发生相反的情况，即Thread-1获得了Thread-0等待的锁，我们就创建了一个死锁。

## 4. 死锁诊断示例

让我们看一些简单的代码来说明以上所有内容。我们将创建一个具有两个线程和两个ReentrantLock对象的死锁场景：

```java
public class Application {

    public static void main(String[] args) throws Exception {
        ReentrantLock firstLock = new ReentrantLock(true);
        ReentrantLock secondLock = new ReentrantLock(true);
        Thread first = createThread("Thread-0", firstLock, secondLock);
        Thread second = createThread("Thread-1", secondLock, firstLock);

        first.start();
        second.start();

        first.join();
        second.join();
    }
}
```

我们的main()方法创建了两个ReentrantLock对象。第一个线程Thread-0使用firstLock作为其主锁，使用secondLock作为其辅助锁。

我们将为Thread-1做相反的事情。具体来说，我们的目标是通过让每个线程获取其主锁并在尝试获取其辅助锁时挂起来生成死锁。

createThread()方法使用各自的锁生成我们的每个线程：

```java
public static Thread createThread(String threadName, ReentrantLock primaryLock, ReentrantLock secondaryLock) {
    return new Thread(() -> {
        primaryLock.lock();

        synchronized (Application.class) {
            Application.class.notify();
            if (!secondaryLock.isLocked()) {
                Application.class.wait();
            }
        }

        secondaryLock.lock();

        System.out.println(threadName + ": Finished");
        primaryLock.unlock();
        secondaryLock.unlock();
    });
}
```

为了确保每个线程在另一个线程尝试之前锁定其primaryLock，我们在同步块中使用isLocked()等待它。

运行此代码将挂起并且永远不会打印完成的控制台输出。如果我们运行jstack，我们将看到以下内容：

```bash
"Thread-0" #12 prio=5 os_prio=0 tid=0x0000027e1e31e800 nid=0x7d0 waiting on condition [0x000000a29acfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e182558>

   Locked ownable synchronizers:
        - <0x000000076e182528>


"Thread-1" #13 prio=5 os_prio=0 tid=0x0000027e1e3ba000 nid=0x650 waiting on condition [0x000000a29adfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e182528>

   Locked ownable synchronizers:
        - <0x000000076e182558>
```

我们可以看到Thread-0等待0x000000076e182558而Thread-1等待0x000000076e182528。同时，我们可以在各自线程的Locked ownable synchronizers中找到这些句柄。**基本上，这意味着我们可以看到我们的线程正在等待哪些锁以及哪些线程拥有这些锁**。这有助于我们排除包括死锁在内的并发问题。

需要注意的重要一点是，如果我们使用ReentrantReadWriteLock.ReadLock作为同步器而不是ReentrantLock，我们将不会在线程转储中获得相同的信息。**只有ReentrantReadWriteLock.WriteLock显示在同步器列表中**。

## 5. 总结

在本文中，我们讨论了线程转储中显示的锁定的可拥有同步器列表的含义，我们如何使用它来解决并发问题，还看到了一个示例场景。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-4)上获得。