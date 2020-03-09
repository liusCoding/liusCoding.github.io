---
layout: post
title:  Java并发-内存模型JMM（二）
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---


1.线程与JVM
2.JVM内存模型与Java内存模型的区别
3.硬件内存架构与Java内存模型
4.Java内存模型对并发特征的保证


## 一、基本概念

程序：代码，完成某一件任务，代码序列（静态的概念）
进程：程序的一次运行（动态的概念）
线程：一个进程可能包含一个或多个线程（cpu分配资源的独立单位）


## 二、JVM与线程

JVM什么时候启动？

答： java类被调用时

JVM线程 -->  其他的线程（main）   线程在jvm中

**JVM内存区域** 
![webp](https://upload-images.jianshu.io/upload_images/10006199-a4108d8fb7810a71.jpeg?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

方法区：
类的所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在这里定义。简单来说，所有定义的方法的信息都保存在该区域，静态变量+常量+类信息（构造方法/接口定义）+运行时常量池都存在方法区中，虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做（Non-Heap）非堆，目的应该为了和Java的堆区分开（Jdk1.8以前hotspot虚拟机叫永久代，持久代，jdk时叫元空间）

堆：
- 新生代（Young Generation）
  - 类出生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器回收，结束生命。
  - 新生代分为两个部分：
    - 伊甸区（Eden space）和幸存者区（Survivor space），所有的类都是在Eden space被new出来的。幸存区又分为From区和To区。当Eden区的空间用完时，程序又需要创建新的对象，JVM的垃圾回收器将Eden区进行垃圾回收（Minior  GC），将Eden区中不再被其它对象引用的对象进行销毁。然后将Eden区中剩余的对象移到From  Surivor区。如From  Survior区也满了，再对该区进行垃圾回收，然后移动到To  Survior区。

- 老年代
  - 新生代经过多次GC仍然存活的对象移动到老年区，若老年代也满了，这时候发生Major GC（也可以叫 Full GC），进行老年区的内存清理。若老年区执行了Full GC之后依然无法进行对象的保存，就会抛出OOM（OutOfMemoryError） 异常。

- 元空间
  - 在jdk1.8以后，元空间替代了永久代，它是对JVM规范中方法区的实现，区别在于元数据区不再虚拟机中，而是用的本地内存，永久代在虚拟机中，永久代的逻辑结构上也属于堆，但是物理上不属于。
  - 为什么移除永久代？
    - 参考官方解释：http://openjdk.java.net/jeps/122
    - 大概意思是移除永久代是为融合HotSpot与 JRockit而做出的努力，因为JRockit没有永久代，不需要配置永久代。

虚拟机栈：
Java线程执行方法的内存模型，一个线程对应一个栈，每个方法在执行的同时都会创建一个栈帧（用于存储局部变量表，操作数栈，动态链接，方法出口等信息）不存在垃圾回收问题，只要线程一结束该栈就会释放，生命周期和线程一致

本地方法栈：
 跟虚拟机的差不多
 
程序计算器：
就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址，也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常的小的内存空间，几乎可以忽略不计。


## 三、Java内存模型

全称 Java Memory model 

![webp](https://upload-images.jianshu.io/upload_images/4222138-96ca2a788ec29dc2.png?imageMogr2/auto-orient/strip|imageView2/2/w/579/format/webp)

JMM定义了Java 虚拟机(JVM)在计算机内存(RAM)中的工作方式。JVM是整个计算机虚拟模型，所以JMM是隶属于JVM的。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

主内存：线程共享的内存

工作内存：私有信息，基本的数据类型，直接分配到工作内存，引用的地址存放在工作内存，引用的对象放在堆中。

工作方式：
- 1.线程修改私有数据，直接在工作空间修改
- 2.线程修改共享数据，把数据copy到工作空间去，在工作空间修改，修改完成之后，在刷新到内存中去。

## 四、硬件内存与Java内存模型

硬件内存架构

![webp](https://upload-images.jianshu.io/upload_images/4222138-49df5535c55287c4.png?imageMogr2/auto-orient/strip|imageView2/2/w/562/format/webp)

该内存架构带来的问题：

1.CPU缓存的一致性问题：并发处理的不同步
2.解决方案：
- 总线加锁 缺点：降低CPU的吞吐量
- 缓存上的一致性协议（MESI）
  - 当cpu在cache中操作数据时，如果该数据是共享变量，数据在cache读到CPU的寄存器中，进行新修改，并更新内存数据cache， LINE置无效，其他的cpu就从内存中读数据。


## 五、Java内存模型的必要性

java内存模型的作用：规范内存数据和工作空间数据的交互

## 六、并发编程的三个重要特性

1. 原子性：不可分割  x=1
2. 可见性：线程只能操作自己工作空间中的数据
3. 有序性：程序中的顺序不一定就是执行的顺序

编译重排序，指令重排序，提高效率

as-if seria:单线程中重排后不影响执行的结果

多线程：
happens-before

**JMM对三个特性的保证**

1.JMM与原子性
  - a. X=10  写  原子性   如果是私有数据具有原子性，如果是共享数据没原子性（读写）
  - b. 表达式：y=x 没有原子性
    -  把数据X读到工作空间（原子性）
    -  把X的值写到Y（原子性）

  - c. i++ 表达式 没有原子性
    -  读i到工作空间
    -   +1；
    -  刷新结果到内存 

多个原子性的操作合并到一起没有原子性
保证方式：
  synchronized关键字，JUC Lock的lock
  
2. JMM与可见性

volatile关键字 在JMM 模型实现MESI协议
synchronized 加锁
JUC Lock的lock

3. JMM与有序性

volatile关键字
synchronized关键字

happens-before原则：
- 程序的次序原则
- 锁定原则：后一次加锁必须等前一次解锁
- volatile原则：不允许重排序
- 传递原则：A--B---C 推导 A -- C


参考文章：
[深入理解JVM-内存模型（jmm）和GC - 简书](https://www.jianshu.com/p/76959115d486)

[并发概念模型：JMM(JAVA内存模型) - Dorami - 博客园](https://www.cnblogs.com/wait-pigblog/p/9372545.html)