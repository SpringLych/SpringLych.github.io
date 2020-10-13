---
title: Java并发学习-三
date: 2019-03-28 15:50:02
tags: [Java,并发]
categories: [Java]
---

Java中的锁，本文主要介绍synchronized关键字和ReentrantLock

<!--more-->

## 锁的分类

悲观锁：对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。**Synchronize关键字和Lock的实现类都是悲观锁**

乐观锁：自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。Java中常用**CAS算法实现乐观锁**。

* Synchronized。JVM实现
* 显式Lock。JDK实现

## 线程安全

首先需要理解线程安全的两个方面：执行控制和内存可见。

**执行控制**的目的是控制代码执行（顺序）及是否可以并发执行。

**内存可见**控制的是线程执行结果在内存中对其它线程的可见性。根据Java内存模型的实现，线程在具体执行时，会先拷贝主存数据到线程本地（CPU缓存），操作完成后再把结果从线程本地刷到主存。

## synchronized

基本性质：

* 互斥锁。一次只允许一个线程进入被锁 的代码块。
* 使用对象的内置锁将代码块锁定。Java中每个对象都有一个**内置锁**
* 保证线程的**原子性**。被保护的代码块是一次被执行的，没有任何线程会同时访问
* 保证线程的**可见性**。当执行完synchronized之后，修改后的变量对其他的线程是可见的

**通过内置锁实现原子性和可见性**。

synchronized是Java提供的原子性内置锁，每个对象都可以把它当做锁来使用，称为内部锁，监视器锁。synchronized代码块自动获取内部锁，。

**synchronized**关键字解决的是执行控制的问题，它会阻止其它线程获取当前对象的监控锁，这样就使得当前对象中被synchronized关键字保护的代码块无法被其它线程访问，也就无法并发执行。

更重要的是，synchronized还会创建一个内存屏障，内存屏障指令保证了所有CPU操作结果都会直接刷到主存中，从而保证了操作的内存可见性，同时也使得先获得这个锁的线程的所有操作，都happens-before于随后获得这个锁的线程的操作。

### 用法

几种用途：

* 修饰代码块。被修饰的代码块称为同步语句块，作用范围是大括号括起来的代码，作用对象是调用这个代码块的对象。
* 修饰普通方法。被修饰的方法为同步方法，作用是整个方法，作用对象是调用这个方法的对象。s
* 修饰静态方法。作用于整个静态方法，作用对象是这个类的所有对象。
* 修饰类：作用是synchronized括号中的部分，对象是这个类中所有对象。

#### 修饰代码块

作用于同一个对象

```java
public class BlockDemo implements Runnable{
    private static int count = 0;

    @Override
    public void run() {
        synchronized (this){
            for (int i = 0; i < 5; i++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ": " + count++);
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        BlockDemo blockDemo = new BlockDemo();
        Thread thread1 = new Thread(blockDemo, "thread1");
        Thread thread2 = new Thread(blockDemo, "thread2");
        thread1.start();
        thread2.start();
    }
}
/**
thread1: 0
thread1: 1
thread1: 2
thread1: 3
thread1: 4
thread2: 5
thread2: 6
thread2: 7
thread2: 8
thread2: 9
**/
```

两个并发线程(thread1和thread2)访问同一个对象(blockDemo)中的synchronized代码块，同一时刻只能有一个线程执行，必须等到一个线程执行完另一个线程才执行。

修改

```java
Thread thread1 = new Thread(new BlockDemo(), "thread1");
Thread thread2 = new Thread(new BlockDemo(), "thread2");
thread1.start();
thread2.start();
/**
thread1: 0
thread2: 1
thread1: 2
thread2: 3
thread2: 4
thread1: 5
thread2: 6
thread1: 7
thread1: 8
thread2: 9
**/
```

两把锁分别锁定两个对象，而这两把锁是互不干扰的，不形成互斥，所以两个线程可以同时执行

#### 修饰方法

作用于同一个对象

```java
public synchronized void func () {
    // ...
}
```

synchronized关键字不能继承。 

#### 静态方法

```java
public synchronized static void fun() {
    // ...
}
```

态方法是属于类的而不属于对象的。同样的，synchronized修饰的静态方法锁定的是这个类的所有对象。

#### 修饰类

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。



### 原理

Java虚拟机中的同步基于进入和退出monitor对象实现，无论是显式同步哈市隐式同步都是如此。

**Monitor**：一个同步工具或同步机制。每个Java对象都有一个看不见的锁，称为内部锁或Monitor锁。

synchronized底层是是**通过进入和退出monitor对象实现，对象有自己的对象头，存储了很多信息，其中一个信息标示是被哪个线程持有**。

### 优化

锁有四种状态：无锁，偏向锁，轻量级锁，重量级锁。锁可以升级，但是是单向的，只能从低到高级，不会出现锁的降级。

#### 无锁

没有对资源进行锁定，但是同时只有一个线程能修改成功。修改会在循环内进行，线程不断尝试修改共享资源，成功就退出，失败继续尝试。**CAS即是无锁的实现**。

#### 偏向锁

同一段代码一直被一个线程访问，那该线程自动获取锁，降低所得获取代价。目标就是在只有一个线程同步代码块时能够提高性能。

可以通过JVM参数关闭偏向锁：-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级锁状态。

#### 轻量级锁

当锁是偏向锁时，被另外的线程访问，偏向锁升级为轻量级锁，其他线程通过自旋的形式获取锁，不会阻塞。

#### 重量级锁

当自旋超过一定的次数，升级为重量级锁

#### 可重入性

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁

## 与volatile比较

1. volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读。synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2. volatile仅能够修饰变量，synchronized可以修饰变量，方法，类级别
3. volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
4. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
5. volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化

对于工作内存与主内存同步延迟现象导致的可见性问题，可以使用synchronized关键字或者volatile关键字解决，它们都可以使一个线程修改后的变量立即对其他线程可见。对于指令重排导致的可见性问题和有序性问题，则可以利用volatile关键字解决，因为volatile的另外一个作用就是禁止重排序优化。

## ReentrantLock

### 使用

```java
public class LockExample {
    private Lock lock = new ReentrantLock();

    public void function() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        LockExample lockExample = new LockExample();
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(lockExample::function);
        service.execute(lockExample::function);
        
    }
}
// 结果
// 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 
```

### 原理

ReentrantLock是基于AQS（AbstractQueuedSynchronizer）实现的，AQS的基础是CAS。

### 与synchronized比较

* synchronize是JVM实现，Lock是JDK实现。
* 性能差别不大
* synchronize的锁和Lock都是非公平锁，Lock可以实现公平锁
* 一个 ReentrantLock 可以同时绑定多个 Condition 对象。

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 参考

* [掘金 - Java锁机制了解一下](https://juejin.im/post/5adf14dcf265da0b7b358d58)
* [CS-Notes](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java 并发.md#五互斥同步)
* [美团技术团队 - 不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)
* [深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483)
* 《Java并发编程的艺术》
* [CSDN - 全面理解Java内存模型(JMM)及volatile关键字
* [Java中Synchronized的用法](<https://blog.csdn.net/luoweifu/article/details/46613015>)