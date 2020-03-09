---
layout: post
title:  Java并发-volatile 关键字实现原理（四）
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---

## 一、认识volatile关键字

**程序举例**

用一个线程读数据，一个线程改数据

```java
public class ReaderAndUpdater {
    public static  int MAX = 50;
    static  volatile int initValue = 0;

    public static void main(String[] args) {
        new Thread(()->{
            int localValue = initValue;
            while (localValue < MAX){
                if(localValue!=initValue){
                    System.out.println("Reader:"+initValue);
                    localValue=initValue;
                }
            }
        },"Reader").start();


        new Thread(()->{
            int localValue = initValue;
            while (localValue<MAX){
                System.out.println("Updater:"+(++localValue));
                initValue = localValue;

                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"Updater").start();
    }
}


```
出现的问题，存在数据的不一致问题
![1580808366102-c69ed9f2-ab60-4863-9c29-7dcfeee4a83f.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1580808366102-c69ed9f2-ab60-4863-9c29-7dcfeee4a83f.png)

解决方法：加volatile关键字
![1580808515162-18500314-7597-42a0-b671-43b7e42e1385.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1580808515162-18500314-7597-42a0-b671-43b7e42e1385.png)

## 二、机器硬件CPU与JMM

参考文章：[CPU Cache - 留一日白 - 博客园](https://www.cnblogs.com/lyrb/p/10718935.html)

1.Cpu Cache模型
![1593920-20190416194930030-166162338.png](https://img2018.cnblogs.com/blog/1593920/201904/1593920-20190416194930030-166162338.png)

![1593920-20190416174053892-1557587328.png](https://img2018.cnblogs.com/blog/1593920/201904/1593920-20190416174053892-1557587328.png)

2.cpu缓存的一致性问题
![1580808884293-66652b20-367f-41f5-9428-46813e2101bc.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1580808884293-66652b20-367f-41f5-9428-46813e2101bc.png)

**解决方案**
a. 总线加锁（粒度太大）
b. MESI
- 1.读操作：不做任何事情，将内存中的数据读到Cache中。
- 2.写操作，发出信号通知其它cpu将该变量的CacheLine置为无效，其它的cpu要访问这个变量的时候，只能从内存中获取。

c.Java内存模型
![1593920-20190421223248015-2062686545.png](https://img2018.cnblogs.com/blog/1593920/201904/1593920-20190421223248015-2062686545.png)

- 1.主内存的数据所有线程都可以访问。
- 2.每个线程都有自己的本地内存。
- 3.本地内存的数据：局部变量、内存的副本
- 4.线程不能直接修改内存中的数据，只能读到本地内存来修改，修改完成后刷新到主内存。


## 三、volatile关键字的语义分析

volatile作用：让其它线程能够马上感知到某一线程对某个变量的修改。

1.保证可见性
对共享变量的修改，其它线程马上能感知到
不能保证原子性 读、写（i++）

2.保证有序性
重排序（编译阶段，指令优化阶段）
输入程序的代码顺序并不是实际代码执行的顺序
重排序后对单线程没有影响，对多线程有影响
volatile 
Happens-before原则

volatile原则：
- 对于volatile修饰的变量：
- a.volatile之前的代码不能调整到它的后面
- b.volatile之后的代码不能调整到它的前面
- c.位置不变化

3.volatile的原理实现机制

轻量级锁
反编译
volatile int a;
LocK:a;

## 四、volatile的使用场景

1.状态标志（开关模式）

```java
public class ShutDowsnDemmo extends Thread{
    private volatile boolean started=false;

    @Override
    public void run() {
        while(started){

        }
    }
    public void shutdown(){
        started=false;
    }
}


```

2.双重检查锁定（double-checked-locking）

```java
public class Singleton {
    private volatile static Singleton instance;
    public static Singleton getInstance(){
        if(instance==null){
            synchronized (Singleton.class){
                instance=new Singleton();
            }
        }
        return instance;
    }
}


```
3.需要利用顺序性。

## 五、volatile和synchronized的区别

1.使用上的区别

volatile只能修饰变量，synchronized只能修饰方法和语句块。

2.对原子性的保证

synchronized可以保证原子性，volatile不能保证原子性

3.对可见性的保证

都可以保证可见性，但实现原理不同

volatile对变量加了lock，synchronized使用monitorEnter和monitorExit  

4.对有序性的保证

volatile能保证有序性，synchronized可以保证有序性，但是代价太大（重量级），并发退化到串行

5.其他

synchronized 引起阻塞
volatile 不会引起阻塞