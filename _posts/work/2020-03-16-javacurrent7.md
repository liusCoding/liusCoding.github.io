---
layout: post
title:  Java并发- 并发编程之AQS的原理(七)
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---
<a name="yLVWK"></a>
### 一.AQS原理
AQS全称为AbstractQueuedSynchronizer，它提供了一个FIFO队列，可以看成是一个用来实现锁以及其它涉及到同步功能的核心组件，常见的有：ReenTrantLock、CountDownLatch等。<br />
<br />AQS是一个抽象类，主要是通过继承的方式来使用，它本身没有实现任何的同步接口，仅仅是定义了同步状态的获取以及释放的方法来提供自定义的同步组件。<br />![](https://cdn.nlark.com/yuque/0/2020/webp/440247/1584350866619-10ffbdba-4cd4-4966-bd3d-87f7c5371e17.webp#align=left&display=inline&height=590&originHeight=590&originWidth=504&size=0&status=done&style=none&width=504)<br />
<br />**AQS的主要字段:**
```java
/**
 * 头节点指针，通过setHead进行修改
 */
private transient volatile Node head;

/**
 * 队列的尾指针
 */
private transient volatile Node tail;

/**
 * 同步器状态
 */
private volatile int state;
```


**AQS需要子类实现的方法：**

| 方法名 | 方法描述 |
| :---: | :---: |
| tryAcquire | 以独占模式尝试获取锁，独占模式下调用acquire，尝试去设置state的值，如果设置成功则返回，如果设置失败则将当前线程加入到等待队列，直到其他线程唤醒 |
| tryRelease | 尝试独占模式下释放状态 |
| tryAcquireShared | 尝试在共享模式获得锁，共享模式下调用acquire，尝试去设置state的值，如果设置成功则返回，如果设置失败则将当前线程加入到等待队列，直到其他线程唤醒 |
| tryReleaseShare | 尝试共享模式释放状态 |
| isHeldExclusively | 是否是独占模式，表示是否被当前线程占用 |


![](https://cdn.nlark.com/yuque/0/2020/jpeg/440247/1584349818070-b7a81001-2d98-4308-abb8-31f27d34ae64.jpeg#align=left&display=inline&height=401&originHeight=401&originWidth=852&size=0&status=done&style=none&width=852)<br />head节点是队列初始化的时候一个节点，只表示位置，不代表实际的等待线程。head节点之后的节点就是获取锁失败进入等待队列的线程。接下来，我们打开AQS源码，看下Node节点都有哪些关键内容：

```java
static final class Node {
        /** 共享模式 */
        static final Node SHARED = new Node();

        /**独占模式 */
        static final Node EXCLUSIVE = null;

        /** 节点状态值，表示节点已经取消 */
        static final int CANCELLED =  1;

        /** 节点状态值，在当前节点释放或者取消的时候，会唤醒下一个节点 */
        static final int SIGNAL    = -1;

        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;

        /**
         * 这个值是在共享锁的时候会用到，唤醒了一个节点，会尝试唤醒下一个节点，
         * 如果当前节点未阻塞（阻塞前就获得了锁）,那么当前节点的状态会被设置成-3
         */
        static final int PROPAGATE = -3;

        /**
         * 等待状态
         */
        volatile int waitStatus;

        /**
         * 前驱节点
         */
        volatile Node prev;

        /**
         * 后继节点
         */
        volatile Node next;

        /**
         * 等待的线程
         */
        volatile Thread thread;

        /**
         *此处可忽略，主要是模式的判断
         */
        Node nextWaiter;

    }
```

<br />通过上面的内容我们可以看到waitStatus其实是有5个状态的，虽然这里面0并不是什么字段，但是他是waitStatus状态的一种，表示不是任何一种类型的字段，上面也讲解了关于AQS中子类实现的方法，AQS提供了独占模式和共享模式两种
<a name="unkvd"></a>
### 二.自定义AQS锁

**自定义锁**

```java
public class MyLock implements Lock {

    private DiyLock diyLock = new DiyLock();
    static class DiyLock extends AbstractQueuedSynchronizer{

        /**
         * Creates a new {@code AbstractQueuedSynchronizer} instance
         * with initial synchronization state of zero.
         */
        protected DiyLock() {
            super();
        }


        //获取锁
        @Override
        protected boolean tryAcquire(int arg) {

            //1.获取状态
            int state = getState();

            if(state == 0){
                //利用CAS原理修改state
                if(compareAndSetState(0,arg)){
                    //可以修改，则设置当前线程占有资源
                    setExclusiveOwnerThread(Thread.currentThread());
                    return true;
                }
            }else if(getExclusiveOwnerThread() == Thread.currentThread()){
                //当前线程重入
                setState(getState() + arg);
                return true;
            }
            //获取失败
            return false;
        }

        //释放锁
        @Override
        protected boolean tryRelease(int arg) {
            int state = getState() -arg;

            //判断释放后是否为0
            if(state == 0){
                //释放当前占有线程
                setExclusiveOwnerThread(null);
                setState(state);
                return true;
            }
            setState(state);//重入性问题
            return false;
        }

        public Condition newCondtionObject(){
            return new ConditionObject();

        }

    }

    @Override
    public void lock() {
        diyLock.acquire(1);
    }


    @Override
    public void lockInterruptibly() throws InterruptedException {
        diyLock.acquireInterruptibly(1);
    }


    @Override
    public boolean tryLock() {
        return diyLock.tryAcquire(1);
    }


    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return diyLock.tryAcquireNanos(1,unit.toNanos(time));
    }


    @Override
    public void unlock() {
        diyLock.release(1);
    }


    @Override
    public Condition newCondition() {
        return diyLock.newCondtionObject();
    }
}

```

**测试自定义锁：**

```java
public class TestMyLock {
    private MyLock lock = new MyLock();

    private int m=0;

    public void increment(){
        lock.lock();
        m++;
        lock.unlock();
    }

    public static void main(String[] args) throws InterruptedException {
        TestMyLock testMyLock = new TestMyLock();

        for (int i = 0; i < 20000; i++) {
            new Thread(() -> {
                testMyLock.increment();
            }).start();
        }

        TimeUnit.SECONDS.sleep(2);
        System.out.println("m="+testMyLock.m);
    }

}
```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584497414121-38dc5d63-228b-499a-afa3-99f609dcb79a.png#align=left&display=inline&height=164&name=image.png&originHeight=164&originWidth=432&size=55900&status=done&style=none&width=432)

测试自定义锁的重入性：

```java
public class TestMyLock2 {
    private MyLock myLock = new MyLock();

    public void entry1(){
        myLock.lock();
        System.out.println("第一次加锁");
        entry2();
        myLock.unlock();
        System.out.println("释放第一次加锁");
    }

    public void entry2(){
        myLock.lock();
        System.out.println("第二次加锁");
        myLock.unlock();
        System.out.println("释放第二次加锁");
    }

    public static void main(String[] args) {
        TestMyLock2 test  = new TestMyLock2();

        new Thread(() ->{
            test.entry1();
        }).start();
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584498550527-d4374e35-ea4b-4f9b-9f19-488be78fb77c.png#align=left&display=inline&height=120&name=image.png&originHeight=120&originWidth=268&size=26548&status=done&style=none&width=268)

**测试ReentrantLock锁的重入性：**

```java
    private ReentrantLock lock=new ReentrantLock();
    public void entry1(){
        lock.lock();
        System.out.println("第一次加锁");
        entry2();
        lock.unlock();
        System.out.println("释放第一次加锁");
    }

    public void entry2(){
        lock.lock();
        System.out.println("第二次加锁");
        lock.unlock();
        System.out.println("释放第二次加锁");
    }

    public static void main(String[] args) {
        TestMyLock2 test  = new TestMyLock2();

        new Thread(() ->{
            test.entry1();
        }).start();
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584498637612-80f3041b-9c26-4395-8280-ab00774dcbdd.png#align=left&display=inline&height=196&name=image.png&originHeight=196&originWidth=385&size=62407&status=done&style=none&width=385)

<a name="3aI2r"></a>
### 三.AQS并发工具
<a name="PYMEI"></a>
#### 1.CountDownLatch

<br />**概念：**<br />CountDownLatch可以使一个或多个线程等待其他线程各自执行完毕后再执行。<br />
<br />CountDownLatch定义了一个计数器，和一个阻塞队列，当计数器的值递减0之前，阻塞队列里面的线程处于挂起状态，当计数器递减到0时会唤醒阻塞队列所有线程，这里的计数器是一个标志，可以表示一个任务一个线程，也可以表示一个倒计时器，CountDownLatch可以解决那些一个或者多个线程执行之前依赖于某些必要的前提业务先执行的场景。<br />
<br />**常用的方法说明：**
```java
CountDownLatch(int count);//构造方法， 创建一个值count的计数器

await();//阻塞当前线程，将当前线程加入阻塞队列。

await(long timeout,TimeUnit unit);//在timeout的时间之内阻塞当前线程，时间一过则当前线程可以执行

countDown();//对计数器进行递减1操作，当计数器递减至0时，当前线程会去唤醒阻塞队列的所有线程。
```


**使用CountDownLatch模拟航空公司查询机票：**


```java
public class TestCountDownLatch {

    private static List<String> airCompany = Stream.of("东方航空","南方航空","中国航空").collect(toList());
    private static List<String> fightList = new ArrayList<>();

    public static void main(String[] args) throws InterruptedException {
        String origin = "北京";
        String dest = "上海";

        Thread[] threads = new Thread[airCompany.size()];
        CountDownLatch latch = new CountDownLatch(airCompany.size());

        for (int i = 0; i < threads.length; i++) {
            String name = airCompany.get(i);

            threads[i] = new Thread(() ->{
                System.out.printf("%s查询从   %s到%s的 机票\n",name,origin,dest);
                //随机产生票数
                int val = new Random().nextInt(10);
                try {
                    TimeUnit.SECONDS.sleep(val);
                    fightList.add(name  + "---------" + val);
                    System.out.printf("%s公司查询成功！\n",name);
                    latch.countDown();
                }catch (Exception e){
                    e.printStackTrace();
                }
            });
            threads[i].start();
        }

        latch.await();

        System.out.println("====================查询结果如下====================");
        fightList.forEach(System.out::println);
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584501020801-4b0a30e4-a2ec-42da-940a-2f0363504658.png#align=left&display=inline&height=291&name=image.png&originHeight=291&originWidth=494&size=105278&status=done&style=none&width=494)


<a name="PEppN"></a>
#### 2.CyclicBarrier
现实生活中我们经常会遇到这样的情景，在进行某个活动前需要等待人全部都齐了才开始。例如吃饭时要等全家人都上座了才动筷子，旅游时要等全部人都到齐了才出发，比赛时要等运动员都上场后才开始。

在JUC包中为我们提供了一个同步工具类能够很好的模拟这类场景，它就是CyclicBarrier类。利用CyclicBarrier类可以实现一组线程相互等待，当所有线程都到达某个屏障点后再进行后续的操作。

CyclicBarrier字面意思是“可重复使用的栅栏”，CyclicBarrier 相比 CountDownLatch 来说，要简单很多，其源码没有什么高深的地方，它是 ReentrantLock 和 Condition 的组合使用。

**实例代码,模拟跑步比赛：**

```java
public class TestCyclicBarrier {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(4);

        Thread[] racer = new Thread[4];

        for (int i = 0; i < 4; i++) {

           racer[i] = new Thread(()->{
                try {
                    TimeUnit.SECONDS.sleep(new Random().nextInt(10));
                    System.out.println(Thread.currentThread().getName() +"准备好了");
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println("选手"+Thread.currentThread().getName()+"起跑");
            },"racer["+i+"]");
            racer[i].start();
        }
    }
}
```

执行结果：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584502145544-7f46ff79-bfde-49cb-a689-84aade4b6c50.png#align=left&display=inline&height=263&name=image.png&originHeight=263&originWidth=443&size=89301&status=done&style=none&width=443)

<a name="ZJ49J"></a>
#### 3.Semaphore 
  适合资源有限的场景

  Semaphore是一种在多线程环境下使用的设施，该设施负责协调各个线程，以保证它们能够正确、合理的使用公共资源的设施，也是操作系统中用于控制进程同步互斥的量。Semaphore是一种计数信号量，用于管理一组资源，内部是基于AQS的共享模式。它相当于给线程规定一个量从而控制允许活动的线程数。


**工作原理：**<br />     以一个停车场是运作为例。为了简单起见，假设停车场只有三个车位，一开始三个车位都是空的。这时如果同时来了五辆车，看门人允许其中三辆不受阻碍的进入，然后放下车拦，剩下的车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开两辆，则又可以放入两辆，如此往复。这个停车系统中，每辆车就好比一个线程，看门人就好比一个信号量，看门人限制了可以活动的线程。假如里面依然是三个车位，但是看门人改变了规则，要求每次只能停两辆车，那么一开始进入两辆车，后面得等到有车离开才能有车进入，但是得保证最多停两辆车。对于Semaphore类而言，就如同一个看门人，限制了可活动的线程数。

**semaphore主要方法：**

```java
Semaphore(int permits):构造方法，创建具有给定许可数的计数信号量并设置为非公平信号量。

Semaphore(int permits,boolean fair):构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量。

void acquire():从此信号量获取一个许可前线程将一直阻塞。相当于一辆车占了一个车位。

void acquire(int n):从此信号量获取给定数目许可，在提供这些许可前一直将线程阻塞。比如n=2，就相当于一辆车占了两个车位。

void release():释放一个许可，将其返回给信号量。就如同车开走返回一个车位。

void release(int n):释放n个许可。

int availablePermits()：当前可用的许可数。
```

**实例说明：**

```java
public class TestSemaphore {

    public static void main(String[] args) {
        /**
         * 创建信号量
         */
        Semaphore sp = new Semaphore(3);

        Thread[] car = new Thread[5];

        for (int i = 0; i < 5; i++) {
            car[i] = new Thread(() -> {
                //请求许可  请求进入停车场
                try {
                    sp.acquire();
                    System.out.println(Thread.currentThread().getName() + "可以进入停车场");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                //使用资源
                int val = new Random().nextInt(10);
                try {
                    TimeUnit.SECONDS.sleep(val);
                    System.out.println(Thread.currentThread().getName()+"停留了" + val  + "秒");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                //释放资源
                sp.release();
                System.out.println(Thread.currentThread().getName() +"离开停车场");
            },"car["+i+"]");

            car[i].start();

        }
    }

}

```

执行结果：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1584506013398-8b27d196-d820-4140-8e00-7f06787d7446.png#align=left&display=inline&height=401&name=image.png&originHeight=401&originWidth=807&size=298828&status=done&style=none&width=807)
