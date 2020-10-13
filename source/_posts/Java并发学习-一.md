---
title: Java并发学习-一
date: 2019-03-27 21:03:30
tags: [Java,并发]
categories: Java
---

创建线程的三种方式，线程相关方法。

<!--more-->

## 创建线程

三种方式：继承Thread类重写run方法，实现Runnable接口，使用FutureTask方式。

### 继承Thread类重写run方法

```java
class Thread1 extends Thread {
    private String name;
    private int ticket = 20;
    private Random rand = ThreadLocalRandom.current();

    public Thread1(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(name + "运行：" + i);
        }
        try {
            sleep(rand.nextInt(100));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Demo extends Thread {
    public static void main(String[] args) {
        Thread1 thread1 = new Thread1("A");
        Thread1 thread2 = new Thread1("B");
        thread1.start();
        thread2.start();
    }
}
```

### 实现Runnable接口
```java

class DemoByRun implements Runnable {

    private String name;

    public DemoByRun(String name) {
        this.name = name;
    }
    private Random random = ThreadLocalRandom.current();

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(name + "运行：" + i);
        }
        try {
            Thread.sleep(random.nextInt(50));
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

public class Main{
    public static void main(String[] args) {
        new Thread(new DemoByRun("A")).start();
        new Thread(new DemoByRun("B")).start();
    }
}
```

### 使用FutureTask

```java
public class Main5 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        DemoByCallable call = new DemoByCallable();
        FutureTask<String> futureTask = new FutureTask<>(call);
        new Thread(futureTask).start();
        System.out.println(futureTask.get());
    }
}

class DemoByCallable implements Callable<String> {
    @Override
    public String call() {
        return "hello";
    }
}
```

### 继承Thread类的线程同步错误实现

```java
public class Main3 {
    public static void main(String[] args) {
        Ticket3 ticket1 = new Ticket3();
        Ticket3 ticket2 = new Ticket3();
        Ticket3 ticket3 = new Ticket3();
        ticket2.start();
        ticket1.start();
        ticket3.start();
    }
}

class Ticket3 extends Thread {
    private int ticket = 5;

    @Override
    public void run() {
        while (ticket > 0) {
            synchronized (this) {
                if (ticket > 0) {
                    System.out.println(currentThread().getName() + "运行，此时的剩余票数" + this.ticket--);
                }
            }
            try {
                sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### 继承Thread类的线程同步正确实现

```java
public class Main4 {
    public static void main(String[] args) {
        Ticket4 ticket4 = new Ticket4();
        new Thread(ticket4).start();
        new Thread(ticket4).start();
        new Thread(ticket4).start();
    }
}

class Ticket4 extends Thread{
    private int ticket = 6;

    @Override
    public void run() {
        while (ticket > 0){
            synchronized (this){
                System.out.println(currentThread().getName() + "运行，此时的剩余票数为" + -- this.ticket);
            }
            try {
                sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
```



### 比较

实现Runnable接口比继承Thread类所具有的优势：

1）：适合多个相同的程序代码的线程去处理同一个资源

2）：可以避免java中的单继承的限制

3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

两者都有的：

适合多个相同的程序代码的线程去处理同一个资源
增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

## 线程状态的转换

* **新建状态（New）**：新创建了一个线程对象
* **就绪状态（Runnable）**：线程对象创建后，其他线程调用了该对象的 start() 方法，该状态的线程位于可运行的线程池中，变为可运行状态，这个时候，只要获取了 cpu 的执行权，就可以运行，进入运行状态。
* **运行状态（Running）**： 就绪状态的线程从 cpu 获得了执行权之后，便可进入此状态，执行 run() 方法里面的代码。
* **阻塞状态（Blocked）**：阻塞状态是线程因为某种原因失去了 cpu 的使用权，暂时停止运行，一直等到线程进入就绪状态，才有机会转到运行状态，阻塞一般分为下面三种：
  * 等待阻塞 ：运行的线程执行了 wait() 方法， JVM 会把该线程放入线程等待池中，（wait() 会释放持有的锁 ）
  * 同步阻塞：运行的线程在获取对象的同步锁时，如果该同步锁被其他线程占用，这时此线程是无法运行的，那么 JVM 就会把该线程放入锁池中，导致阻塞
  * 其他阻塞：运行的线程执行 sleep() 或者 join() 方法，或者发出了 I/O 请求，JVM 会把该线程置为阻塞状态，当 sleep() 状态超时、join() 等待线程终止或者超时、或者 I/O 处理完毕时，线程会重新进入就绪状态，（注意：sleep() 是不会释放本身持有的锁的）
* 无限期等待：等待其他线程显式地唤醒，否则不分配CPU时间片

* 死亡状态（Dead）：线程执行完了之后或者因为程序异常退出了 run() 方法，结束该线程的生命周期

## 线程间的协作

### wait notify notifyAll

* 三者都是object的方法，不属于thread，只能在synchronized中调用，否则报错IllegalMonitorStateException
* wait方法使线程进入等待状态，并且会**立即释放对象的锁**。如果没有释放锁，其他线程就无法进入对象的同步方法块中，无法使用notify或notifyAll唤醒挂起的线程，造成死锁
* notify会唤醒一个因wait处于阻塞状态的线程，使其进入就绪状态。notify**不会马上释放对象锁**，会等到执行完该同步方法或方法块后再释放
* notifyALl会唤醒等待队列中等待统一资源的全部线程从等待状态退出。优先级最高的执行，也可能是随机执行

**wait()**

```java
public class MyThreadPrint extends Thread {

    private String name;
    private final Object prev;
    private final Object self;

    private MyThreadPrint(String name, Object prev, Object self) {
        this.name = name;
        this.prev = prev;
        this.self = self;
    }

    @Override
    public void run() {
        int count = 10;
        while (count > 0) {
            synchronized (prev) {
                synchronized (self) {
                    System.out.println(name);
                    count--;
                    self.notify();
                }
                try {
                    prev.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Object A = new Object();
        Object B = new Object();
        Object C = new Object();
        MyThreadPrint myA = new MyThreadPrint("A", C, A);
        MyThreadPrint myB = new MyThreadPrint("B", A, B);
        MyThreadPrint myC = new MyThreadPrint("C", B, C);

        try {
            new Thread(myA).start();
            Thread.sleep(100);
            new Thread(myB).start();
            Thread.sleep(100);
            new Thread(myC).start();
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

* Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于**Object.wait()在释放CPU同时，释放了对象锁的控制**。

#### wait和sleep区别

* wait是object的方法，sleep是thread的静态方法

* wait会释放对象锁，sleep不会

共同点： 

1. 他们都是在多线程的环境下，都可以在程序的调用处阻塞指定的毫秒数，并返回。 

2. wait()和sleep()都可以通过interrupt()方法 打断线程的暂停状态 ，从而使线程立刻抛出InterruptedException。 
   如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep /join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。 
   需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用 interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到 wait()/sleep()/join()后，就会立刻抛出InterruptedException 。 
   不同点： 

3. 每个对象都有一个锁来控制同步访问。Synchronized关键字可以和对象的锁交互，来实现线程的同步。 
   sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。 

4. wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用 
   所以sleep()和wait()方法的最大区别是：

   sleep()睡眠时，保持对象锁，仍然占有该锁；而wait()睡眠时，释放对象锁。但是wait()和sleep()都可以通过interrupt()方法打断线程的暂停状态，从而使线程立刻抛出InterruptedException（但不建议使用该方法）。

### Join()

在一个线程中调用另一个线程的join方法，会将当前线程挂起，直到目标线程结束。在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是**主线程需要等待子线程执行完成之后再结束**，这个时候就要用到join()方法了。

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName()+"主线程运行开始!");
		Thread1 mTh1=new Thread1("A");
		Thread1 mTh2=new Thread1("B");
		mTh1.start();
		mTh2.start();
		try {
			mTh1.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		try {
			mTh2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");
	}
}
```

## 线程其他函数

### start与run区别

start()是启动一个新线程。
通过start()方法来启动的新线程，处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，就开始执行相应线程的run()方法，

1. start() 可以启动一个新线程，run()不能
2. start()不能被重复调用，run()可以
3. start()中的run代码可以不执行完就继续执行下面的代码，即进行了线程切换。直接调用run方法必须等待其代码全部执行完才能继续执行下面的代码。
4. start() 实现了多线程，run()没有实现多线程。

### yield():

暂停当前正在执行的线程对象，并执行其他线程。

yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。
结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。

#### sleep()和yield()的区别

* sleep()和yield()的区别):sleep()使当前线程进入停滞状态，所以执行sleep()的线程在指定的时间内肯定不会被执行；yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。  
* sleep 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，yield 方法使当前线程让出 CPU   占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU  的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程  
* 另外，sleep 方法允许较低优先级的线程获得运行机会，但 yield()  方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。在一个运行系统中，如果较高优先级的线程没有调用 sleep 方法，又没有受到 I\O 阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。 

### setPriority(): 更改线程的优先级。

> MIN_PRIORITY = 1  
> NORM_PRIORITY = 5  
> MAX_PRIORITY = 10  

## 参考

[Java多线程学习（吐血超详细总结）](https://blog.csdn.net/evankaka/article/details/44153709)

[Java多线程基础学习](https://www.jianshu.com/p/e87ee54a13a4)

