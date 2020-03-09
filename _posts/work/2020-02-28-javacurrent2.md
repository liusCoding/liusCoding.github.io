---
layout: post
title:  Java并发-Synchronized关键字实现原理（三）
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---


模拟并发叫号程序：
```java
public class TicketDemo extends Thread{

    private static int index = 1;
    private static final int MAX = 50000;

    @Override
    public void run() {
            while(index<=MAX){
                System.out.println(Thread.currentThread().getName()  +"叫到的号码是："+(index++));

        }
    }


    public static void main(String[] args) {
        TicketDemo ticketDemo1 = new TicketDemo();
        TicketDemo ticketDemo2 = new TicketDemo();
        TicketDemo ticketDemo3 = new TicketDemo();
        TicketDemo ticketDemo4 = new TicketDemo();
        TicketDemo ticketDemo5 = new TicketDemo();
        TicketDemo ticketDemo6 = new TicketDemo();

        ticketDemo1.start();
        ticketDemo2.start();
        ticketDemo3.start();
        ticketDemo4.start();
        ticketDemo5.start();
        ticketDemo6.start();
    }
}

```

![1577524785293-9bd0d9ac-847f-44b2-bb13-2dc9b0b8662e.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577524785293-9bd0d9ac-847f-44b2-bb13-2dc9b0b8662e.png)



并发叫号：
static关键字 把变量放到主存，作为共享变量: 并发比较大的时候会出现：跳号、重号、超过最大值

解决方法：
```java
@Override
    public void run() {
          synchronized(this){ 
          while(index<=MAX){
                System.out.println(Thread.currentThread().getName()  +"叫到的号码是："+(index++));
        }
    }
}
```

synchronzied特性：独占性、排它性、可见性

## 一、Synchronized概念
是利用锁的机制来实现同步的。

锁机制有如下两种特性：
- 互斥性：即在同一时间只允许一个线程持有某个对象锁，通过这种特性来实现多线程中的协调特性，这样在同一时间只有一个线程对需同步的代码块（复合操作）进行访问。互斥性我们也往往称为操作的原子性。

- 可见性：必须确保在锁被释放之前，对共享变量所作的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作从而引起的数据不一致

## 二、synchronized的用法

根据修饰对象分类

1. 同步方法
 - a.同步非静态方法
  ```java
  public synchronzied void methodName(){}
  ```
  
- b.同步静态方法
  ```java
  public synchronzied static void methodName(){}
  ```

2. 同步代码块

- a.当前对象
  ```java
  synchronized(this|object)
  ```
- b.字节码对象
  synchronized(类.class)
  
根据获取的锁分类

1. 获取对象锁
synchronized（this|object）{}、修饰非静态方法

在java中，每个对象都会有一个monitor对象，这个对象其实就是java对象的锁，通常会被称为“内置锁”或者“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。

2. 获取类锁
在java中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的Class对象锁，每个类只有一个Class对象，所以每个类只有一个类锁。

在java中，每个对象都会有一个monitor对象，监视器。
- 某一个线程占有这个对象的时候，先判断monitor的计数器是不是0，如果是0还没有线程占有，这个时候占有这个对象，并且对这个对象的monitor+1；如果不为0，表示这个线程已经被其它线程占有，那这个就要等待，当线程释放占有权的时候的时候，monitor-1；

- 同一线程可以对同一对象进行多次加锁，+1，+1，这叫作重入性。

## 三、synchronized 原理分析

实例代码
```java
public class TestSynchronized {
    public TestSynchronized() {
    }

    public static synchronized void accessResources1() {
        try {
            TimeUnit.SECONDS.sleep(2L);
            System.out.println(Thread.currentThread().getName() + "is running");
        } catch (InterruptedException var1) {
            var1.printStackTrace();
        }

    }

    public void accessResources2() {
        synchronized(this) {
            try {
                TimeUnit.SECONDS.sleep(2L);
                System.out.println(Thread.currentThread().getName() + "is running");
            } catch (InterruptedException var4) {
                var4.printStackTrace();
            }

        }
    }

    public void accessResources3() {
        Class var1 = TestSynchronized.class;
        synchronized(TestSynchronized.class) {
            try {
                TimeUnit.MINUTES.sleep(2L);
                System.out.println(Thread.currentThread().getName() + "is running");
            } catch (InterruptedException var4) {
                var4.printStackTrace();
            }

        }
    }

    public static void main(String[] args) {
        TestSynchronized demo1 = new TestSynchronized();

        for(int i = 0; i < 5; ++i) {
            demo1.getClass();
            (new Thread(demo1::accessResources3)).start();
        }

    }
}

```

1. 线程堆栈分析（互斥）

JVM 工具   Jconsole
Waiting  线程已经准备运行
![1577528679451-6b6f1799-d875-4201-842c-611d1ec855fc.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577528679451-6b6f1799-d875-4201-842c-611d1ec855fc.png)

Blocked   线程阻塞
![1577528699202-7c8850a5-c39a-4694-823b-59ff9804cacc.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577528699202-7c8850a5-c39a-4694-823b-59ff9804cacc.png)

2. JVM指令分析

javap -v  类名（TestSynchronized）

**synchronized修饰代码块加锁原理**

![1577598981782-1adfb158-f882-4276-bf0f-0f2e49bfae3f.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577598981782-1adfb158-f882-4276-bf0f-0f2e49bfae3f.png)

![1577602219995-f2be0788-ec20-4660-8850-51c63230d6c0.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577602219995-f2be0788-ec20-4660-8850-51c63230d6c0.png)
monitorenter  互斥入口
- 当moniter为0的时候进入，进入后monitor+1，可以重入，monitor被当前线程占有，其它线程请求时会进入Blocked状态，直到monitor为0的时候才能进入
monitorexit  互斥出口
- monitor计数器-1，为0的时候释放锁

以上是synchronized修饰代码块的加锁的原理，配合monitorenter和monitorexit使用

**synchronized修饰方法加锁原理**

![1577599801364-5a9bf154-520a-4cdf-8843-34e04de4db1d.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577599801364-5a9bf154-520a-4cdf-8843-34e04de4db1d.png)
对方法加锁
加标记：ACC_SYNCHRONIZED

ps:JDK1.6之前Synchronized关键字是重量锁

3. 使用synchronized 注意的问题
- a.与monitor关联的对象不能为空
- b.synchronized作用域太大
- c.多个锁的交叉会导致死锁


## 三、JVM对Synchronized的优化

![1577600237767-bdb0d3f7-2dc4-4bd9-84d6-1251fca4e2a1.png](https://cdn.nlark.com/yuque/0/2019/png/440247/1577600237767-bdb0d3f7-2dc4-4bd9-84d6-1251fca4e2a1.png)

一个对象的实例包含：对象头、实例变量、对齐填充

对象头：是加锁的基础

对象头占4个字节，32位

![1577600671173-8a594389-19f7-4766-a3ab-1c101626de4d.jpeg](https://cdn.nlark.com/yuque/0/2019/jpeg/440247/1577600671173-8a594389-19f7-4766-a3ab-1c101626de4d.jpeg)
JVM 对synchronized的优化：
偏向锁
轻量级锁
重量级锁（等待时间长）

无锁状态：没有加锁

偏向锁：在对象第一次被某一个线程占有的时候，是否偏向锁置1，锁标记位01，写入当前线程号。

采用CAS算法，compare and swap
多次被第一次占有它的线程获取次数多，成功
当其它的线程访问的时候，竞争，如果竞争失败  升级为轻量级锁

和无锁状态时间接近，竞争不激烈的时候适用

轻量级锁：线程有交替适用，互斥性不是很强  锁标记00

重量级锁：强互斥，等待时间长  锁标记10


由于用户线程转成核心线程的过程比较耗时

JVM加入自旋锁

自旋锁：竞争失败的时候，不是马上转化级别，而是执行几次空循环  5次或10次


锁消除：JIT在编译的时候会把不必要的锁去掉









