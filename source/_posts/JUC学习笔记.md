---
title: JUC学习笔记
date: 2024-01-16 20:27:00
tags:
- Java
- 并发编程
- JDK21
- 拾遗
categories:
- 'Java基础知识'
toc: true
cover: https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.4b02j7ej5xa0.png
---

# 简介

从JDK1.5开始，Java提供了`java.util.concurrent`包，在此包中增加了在并发编程中很常用的工具类，并在JDK21中加入了协程（Virtual Thread）。多线程并发作为Java中的重要内容，本文希望从JUC出发，整理相关内容，学习并发编程。

# Java内存模型 / Java Memory Model

> JMM规定所有变量都存储在主内存中，每条线程还有自己的工作内存。线程的工作内存中保存了被线程使用的变量的主内存副本，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的数据。不同线程之间也无法直接访问对方的工作内存中的变量，线程间变量值的传递需要通过主内存来完成。

每条线程都有自己的工作空间，而共享变量存储在共享内存中。线程在运行时会首先将共享内存中的数据读取到自己的工作内存，即在线程的工作内存中复制了一个共享变量的副本，然后对其进行计算，计算完成后线程会将自己工作内存中的这个共享变量副本同步回主内存。线程、工作内存、与主内存的关系如下图所示：

![](https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.4z98ywzthdc0.png)

# 关键字

## volatile

`volatile`变量有两大作用：

1. 保证可见性：指当一个线程修改了共享变量后，另外的线程能立即感知这个变量被修改。
2. 保证有序性：指程序按照代码的先后顺序执行。有时候为了优化性能，编译器会对字节码指令进行重排序，但是能保证重排序后的执行结果与重排序之前是一致的。

### 保证可见性

通过例子来说明：

```java
public class VolatileDemo {

    private static boolean ready;

    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("MyThread is running...");
            while (!ready) ; // 如果ready为false，则死循环
            System.out.println("MyThread is end");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new MyThread().start();
        Thread.sleep(1000);
        ready = true;
        System.out.println("ready = " + ready);
        Thread.sleep(5000);
        System.out.println("main thread is end.");
    }
}
```

代码中定义了一个boolean类型的成员变量`ready`,其默认值为false。在`MyThread`线程中判断如果`ready`为false时则进行死循环。接下来在`main`方法中开启`MyThread`线程，并在睡眠1s后将`ready`修改为true。正常情况下`ready`修改为true后`MyThread`线程中的死循环则会停止，并打印"MyThread is end"。但实际上的运行结果为：

```bash
MyThread is running...
ready = true
main thread is end.
```

可以看见当`ready`被修改为true后，`MyThread`线程依然未结束。通过这一例子也证实了`MyThread`线程中的`ready`副本并没有得到及时的更新。

那么接下来我们将成员变量`ready`使用`volatile`关键字修饰：

```java
public class VolatileDemo {

    private volatile static boolean ready;

    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("MyThread is running...");
            while (!ready) ; // 如果ready为false，则死循环
            System.out.println("MyThread is end");
        }
    }
}
```

再运行看打印日志：

```bash
MyThread is running...
MyThread is end
ready = true
main thread is end.
```

当在主线程中修改了`ready`为true后，`MyThread`线程立即感知了`ready`的变化，并结束了死循环，从这个例子中也可以看出volatile确实能有效的保证多个线程共享变量的可见性。

### 保证有序性

`volatile`保证有序性在开发中有一个很常见的例子，即双重锁校验的**单例模式**下需要使用`volatile`关键字来禁止指令重排序。我们来看代码：

```java
public class DoubleCheckLock {

    private volatile static DoubleCheckLock instance;

    private DoubleCheckLock(){}

    public static DoubleCheckLock getInstance(){
        //第一次检测
        if (instance==null){
            //同步
            synchronized (DoubleCheckLock.class){
                if (instance == null){
                    //多线程环境下可能会出现问题的地方
                    instance = new DoubleCheckLock();
                }
            }
        }
        return instance;
    }
}
```

如果上述代码中没有给`instance`加上`volatile`关键字会怎么呢？

首先我们应该清楚`instance = new DoubleCheckLock();`这一操作并不是一个原子操作，实例化对象的字节指令可以分为如下三步：

1. 分配对象内存：memory = allocate();
2. 初始化对象：instance(memory);
3. instance指向刚分配的内存地址：instance = memory;

而由于编译器的指令重排序，以上指令可能会出现以下顺序：

1. 分配对象内存：memory = allocate();
2. instance指向刚分配的内存地址：instance = memory;
3. 初始化对象：instance(memory);

以优化后的字节码指令来看双重锁校验的代码是否有问题呢？不难发现，如果线程A第一次调用单例方法，在该线程的时间片轮转结束后执行到了优化后的第二个指令，即`instance`被赋值，但是还未被分配初始化对象。此时，线程B抢到了CPU时间片，同时调用了`getInstance`方法，第一次校验就发现`instance`不为null，遂将其返回。在得到这个单例后调用单例的方法，此时必定出现空指针异常。因此，可见指令重排序在多线程并发的情况下是会出现问题的。此时，我们便可以通过`volatile`关键字来禁止编译器的优化，从而避免空指针的出现。

## synchronized

`synchronized` 是 Java 中用于实现线程同步的关键字，它提供了一种独占锁的机制，用于确保多个线程之间的互斥访问共享资源。

`synchronized`可以用来修饰实例方法和静态方法，也可以用来修饰代码块，值的注意的是`synchronized`是一个**对象锁**，因此，无论使用哪一种方法，synchronized都需要有一个锁对象。

### 修饰实例方法

`synchronized`修饰实例方法只需要在方法上加上`synchronized`关键字即可

```java
public synchronized void add(){
       i++;
}
```

此时，`synchronized`加锁的对象就是这个方法**所在实例的本身**。

### 修饰静态方法

`synchronized`修饰静态方法的使用与实例方法并无差别，在静态方法上加上`synchronized`关键字即可

```java
public static synchronized void add(){
       i++;
}
```

此时，`synchronized`加锁的对象为当前静态方法**所在类的Class对象**。

### 修饰代码块

synchronized修饰代码块需要传入一个对象。

```java
public void add() {
    synchronized (this) {
        i++;
    }
}
```

很明显，此时synchronized加锁对象即为传入的这个对象实例。

### synchronized锁优化

为了解决`synchronized`效率低下，在JDK1.6中引入了偏向锁和轻量级锁来优化`synchronized`。此时的synchronized一共存在四个状态：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。锁着锁竞争激烈程度，锁的状态会出现一个升级的过程，即可以从偏向锁升级到轻量级锁，再升级到重量级锁。锁升级的过程是单向不可逆的，即一旦升级为重量级锁就不会再出现降级的情况。

# ReentrantLock

`ReentrantLock`是一种显式锁，需要我们手动编写加锁和释放锁的代码，下面我们来看下`ReentrantLock`的使用方法。

```java
public class ReentrantLockDemo {
    // 实例化一个非公平锁，构造方法的参数为true表示公平锁，false为非公平锁。
    private final ReentrantLock lock = new ReentrantLock(false);
    private int i;

    public void testLock() {
        // 拿锁，如果拿不到会一直等待
        lock.lock();
        try {
            // 再次尝试拿锁(可重入)，拿锁最多等待100毫秒
            if (lock.tryLock(100, TimeUnit.MILLISECONDS))
                i++;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // 释放锁
            lock.unlock(); 
            lock.unlock();
        }
    }
}
```

上述代码中`lock.lock()`会进行拿锁操作，如果拿不到锁则会一直等待。如果拿到锁之后则会执行try代码块中的代码。接下来在try代码块中又通过`tryLock(100, TimeUnit.MILLISECONDS)`方法尝试再次拿锁，此时，拿锁最多会等待100毫秒，如果在100毫秒内能获得锁，则`tryLock`方法返回true，拿锁成功，执行`i++`操作，如果返回false，获取锁失败，`i++`不会被执行。（因为第一次线程已经拿到锁了，由于`ReentrantLock`是可重入，因此，第二次必定能拿到锁。另外，**要注意被`ReentrantLock`加锁区域必须用try代码块包裹，且释放锁需要在finally中来避免死锁**。执行几次加锁，就需要几次释放锁。

## 公平锁与非公平锁

> **公平锁**是指多个线程按照申请锁的顺序来获取锁，线程直接进入同步队列中排队，队列中最先到的线程先获得锁。**非公平锁**是多个线程加锁时每个线程都会先去尝试获取锁，如果刚好获取到锁，那么线程无需等待，直接执行，如果获取不到锁才会被加入同步队列的队尾等待执行。

公平锁和非公平锁各有优缺点，适用于不同的场景：

1. 公平锁的优点在于各个线程平等，每个线程等待一段时间后，都有执行的机会；而它的缺点相较于非公平锁整体执行速度更慢，吞吐量更低。同步队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。
2. 非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程；它的缺点也比较明显，即队列中等待的线程可能一直或者长时间获取不到锁。

## 可重入锁与非可重入锁

> **可重入锁**又名递归锁，是指同一个线程在获取外层同步方法锁的时候，再进入该线程的内层同步方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。**非可重入锁**与可重入锁是对立的关系，即一个线程在获取到外层同步方法锁后，再进入该方法的内层同步方法无法获取到锁，即使锁是同一个对象。

`synchronized`与`ReentrantLock`都属于可重入锁。可重入锁可以有效避免死锁的产生。

## AbstractQueuedSynchronizer

`ReentrantLock`中的几个核心方法的实现都是调用了`Sync`中的相关方法，而`Sync`的主要逻辑在父类`AbstractQueuedSynchronizer`中实现。（待补充）

# CAS



# 参考

- [这一次，彻底搞懂Java内存模型与volatile关键字（系列）](https://juejin.cn/post/6967739352784830494#heading-7)
- [JDK21要来了，协程可以给Java带来什么](https://developer.aliyun.com/article/1290951)
