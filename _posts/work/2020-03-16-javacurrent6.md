---
layout: post
title:  Java并发- 并发编程之CAS的原理(六)
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---

<a name="RqQgw"></a>











### 一.什么是CAS？
CAS（compare And Swap），中文叫比较交换，是一种无锁原子算法。<br />过程是这样：
> 它包含3个参数CAS （V,E,N）,V 表示要更新变量的值，E表示预期值，，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其它线程做了更新，则当前线程则什么都不做，最后，CAS返回当前V的真实值。CAS操作时抱着乐观的态度进行的，它总是认为自己可以成功完成操作。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584324837435-945aff59-d458-4eb3-97cf-7fc8c5747ecb.png#align=left&display=inline&height=353&name=image.png&originHeight=353&originWidth=833&size=35834&status=done&style=none&width=833)


当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余线程均会失败。失败的线程不会挂起，只是被告知失败，并且允许再次尝试，当然也允许实现的线程放弃操作。基于这样的原理，CAS操作即使没有锁，也可以发现其它线程对当前线程的干扰。<br />
<br />与锁相比，使用CAS会使程序看起来更复杂一些，但由于其非阻塞的，它对死锁问题天生免疫，并且，线程间的相互影响也非常小。更为重要的是，使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程间频繁调度带来的开销，因此，它要比基于锁的方式拥有的更优越的性能。<br />
<br />简单的说，CAS需要你额外给出一个期望值，也就是你认为这个变量现在应该是什么样子的，如果变量不是你想想象的那样，那就说明被别的线程修改过了。你就需要重新读取，再次尝试修改就好了。

<a name="z6x6j"></a>
### 二.CAS底层原理
这样归功于硬件指令集的发展，实际上，我们可以使用同步将这两个操作变成原子的，但是这么做就没有意义了。所以我们只能靠硬件来完成，硬件保证一个从语义上看起来需要多次操作的行为只通过一条处理器指令就能完成。这类指令常用的有：

1. 测试并设置（Tetst-and-Set）
1. 获取并增加（Fetch-and-Increment） 
1. 交换（Swap）
1. 比较并交换（Compare-and-Swap） 
1. 加载链接/条件存储（Load-Linked/Store-Conditional）



CPU实现指令有2种方式：

- **1.通过总线锁定来保证原子性**
> 总线锁定其实就是处理器使用了总线锁，所谓总线锁就是使用处理器提供的一个 LOCK# 信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以独占共享内存。但是该方法成本太大。因此有了下面的方式。
> <br />

- **2.通过缓存锁定来保证原子性**
> 所谓 缓存锁定 是指内存区域如果被缓存在处理器的缓存行中，并且在Lock 操作期间被锁定，那么当他执行锁操作写回到内存时，处理器不在总线上声言 LOCK# 信号，而时修改内部的内存地址，并允许他的缓存一致性机制来保证操作的原子性，因为缓存一致性机制会阻止同时修改两个以上处理器缓存的内存区域数据（这里和 volatile 的可见性原理相同），当其他处理器回写已被锁定的缓存行的数据时，会使缓存行无效。
> <br />注意：有两种情况下处理器不会使用缓存锁定。
> 
> 1. 当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行时，则处理器会调用总线锁定。
> 1. 有些处理器不支持缓存锁定，对于 Intel 486 和 Pentium 处理器，就是锁定的内存区域在处理器的缓存行也会调用总线锁定。


<a name="WoKPM"></a>
### 三.举例说明

```java
public class TestCAS {

    /**
     * 定义原子类型Integer  count
     */
    private static AtomicInteger atomicCount = new AtomicInteger(0);

    /**
     *定义Integer count
     */
    private static Integer count = 0;


    /**
     * 进行 count++
     */
    public static void countIncrement(){
        count ++;
    }

    /**
     * 进行 atomicCount++
     */
    public static void atomicCountIncrement(){
        atomicCount.getAndIncrement();
    }

    public static void main(String[] args) throws InterruptedException {

        for (int i = 0; i < 2000; i++) {
            new Thread(() ->{
                TestCAS.countIncrement();
                TestCAS.atomicCountIncrement();
            }).start();
        }

        TimeUnit.SECONDS.sleep(2);
        System.out.println("atomicCount="+atomicCount.get());
        System.out.println("count="+count);
    }

}

```
结果是：![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584326973493-906fe320-586a-43a0-8813-15985c8b0c20.png#align=left&display=inline&height=128&name=image.png&originHeight=128&originWidth=355&size=39610&status=done&style=none&width=355)


<a name="b6xL1"></a>
### 四.源码分析

JUC下的atomic类都是通过CAS来实现的，下面就以AtomicInteger为例来说明CAS的实现，如图：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584327548041-58c3afdf-d52b-42c4-b3cc-01a3d082dabe.png#align=left&display=inline&height=674&name=image.png&originHeight=674&originWidth=932&size=54230&status=done&style=none&width=932)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584327754273-8d3e108d-828a-4f3c-b3f9-88cd195d1fe5.png#align=left&display=inline&height=407&name=image.png&originHeight=407&originWidth=856&size=479522&status=done&style=none&width=856)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584327791937-f5aecea0-4bbd-4df5-8600-21a92ff3ea7d.png#align=left&display=inline&height=332&name=image.png&originHeight=332&originWidth=681&size=162033&status=done&style=none&width=681)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584327808347-476f11ba-4111-4503-884b-9a8be9e109b2.png#align=left&display=inline&height=203&name=image.png&originHeight=203&originWidth=805&size=46222&status=done&style=none&width=805)<br />
<br />Unsafe是CAS的核心类，Java无法直接访问访问底层操作系统，而是本地方法（native）方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe,它提供了硬件级别的原子操作。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584328606007-a8dc7937-699b-47fd-9e02-179ff7f85469.png#align=left&display=inline&height=220&name=image.png&originHeight=220&originWidth=874&size=209327&status=done&style=none&width=874)<br />
<br /> atomicCount.getAndIncrement() 内部调用了unsafe的getAndAddInt方法，在getAndAddInt() 方法中主要是看compareAndSwapInt方法<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584329239001-e4346253-8ebe-4a1d-8b5b-afd254c61207.png#align=left&display=inline&height=91&name=image.png&originHeight=91&originWidth=661&size=97955&status=done&style=none&width=661)<br /> <br />CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，但是在多处理器上实现就有点儿复杂了。

**缓存加锁：**<br />其实针对于上面那种情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可。缓存加锁就是缓存在内存区域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协议来保证原子性。缓存一致性机制可以保证同一个内存区域的数据仅能被一个处理器修改，也就是说当CPU1修改缓存行中的i时使用缓存锁定，那么CPU2就不能同时缓存了i的缓存行<br />
<br />**CAS的缺点：**<br />CAS虽然高效的解决了原子操作，但是还是存在一些缺陷的。<br />
<br />主要表现为三个方面：    <br />1.循环时间太长<br />2.只能保证一个共享变量的原子操作<br />3.ABA问题

循环时间长：
> 如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在           JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。




 只能保证一个共享变量的原子操作:
>         看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。例如读写锁中state的高地位




ABA问题：
> 

>           CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。
