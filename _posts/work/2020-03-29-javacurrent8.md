---
layout: post
title:  Java并发-  用优雅的姿势使用和理解线程池(八)
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---

线程池是管理一组工作线程，通过线程池复用线程


<a name="MLrl6"></a>
### 一.线程池的定义   

<br />管理一组工作线程，通过线程池复用线程。核心的思想就是把宝贵的资源放到一个池子中，每次使用线程都从线程池中获取，用完之后又放回线程池中供其它线程使用。<br />
<br />**使用线程池的好处**

- 降低资源消耗
  - 通过重复利用已创建的线程来降低创建和销毁造成的消耗。
- 提高响应速度
  - 当任务到达时，任务可以不需要等待线程创建就能立即执行。



- 提高线程的可管理性
  - 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。


<br />
<br />

<a name="jVxDE"></a>
### 二.通过Executor框架来线程池
Executor 框架是 Java5 之后引进的，在 Java 5 之后，通过 Executor 来启动线程比使用 Thread 的 start 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免 this 逃逸问题。<br />

> 补充： this逃逸是指构造方法返回之前其它线程就持有该对象的引用，调用尚未构造完全的对象的方法可能引发令人疑惑的错误。


<br />Executor 框架不仅包括了线程池的管理，还提供了线程工厂、队列以及拒绝策略等，Executor 框架让并发编程变得更加简单。<br />
<br />Executor框架创建线程池的方式：
> Executors.newCachedThreadPool
>        创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
> Executors.newFixedThreadPool
>        创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
> Executors.newSingleThreadExecutor 
> 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。


<br />通过查看源码会发现，其实这三种创建的源码都是领用ThreadPollExecutor类实现。<br />

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }

```

<br />这几个核心构造方法参数说明：

- corePoolSize：线程池的核心线程数量
- maximumPoolSize: 线程池最大数量
- keepAliveTime：空闲线程存活时间
- unit：存活时间的单位
- workQueue：用于存放任务的阻塞队列
- handler: 当队列和最大线程池都满了之后的饱和策略。


<br />**线程池执行任务逻辑和线程池参数的关系**<br />**![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4ubmxhcmsuY29tL3l1cXVlLzAvMjAyMC93ZWJwLzQ0MDI0Ny8xNTg1NTU2NzMyNDkwLWY2OGEwYzA5LTZlNDMtNGM3Zi04YWQ2LTcyYTdiNWViZjc3My53ZWJw?x-oss-process=image/format,png#align=left&display=inline&height=311&originHeight=311&originWidth=640&size=0&status=done&style=none&width=640)<br />执行逻辑说明：
> - 判断核心线程是否已满，核心线程数大小和corePoolSize参数有关，未满创建线程执行任务
> - 若核心线程池已满，判断队列是否满，队列是否满和workQueue参数有关，若未满则加入队列中。
> - 若队列已满，判断线程池是否已满，线城池是否满和maximumPoolSize参数有关，若未满创建线程执行任务
> - 若线程池已满，则采用拒绝策略处理无法执行的任务，拒绝策略和handler参数有关



<a name="F5eru"></a>
#### 1.**newCachedThreadPool方法**


```java
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```

<br />CachedThreadPool是一个根据需要缓存创建的线程池
> - corePoolSize=0; 线程线程池的数量为0
> - maximumPoolSize=Integer.MAX_VALUE；可以认为最大线程数是无限的
> - keepAliveTime = 60L
> - unit = TimeUnit.SECOND; 空闲线程的存活时间为60秒
> - workQueue= new SynchronousQueue()


<br />当一个任务提交时，corePoolSize为0则不创建核心线程，SynchronousQueue是一个不存储元素的队列，可以理解为队列永远都是满的，因此最终会创建非核心线程来执行任务。<br />
<br />对于非核心线程60s时将被回收，因为Integer.MAX_VALUE非常大，可以认为是可以无限创建线程的，在资源有限的情况下容易引起OOM异常。

<a name="TtSrk"></a>
#### 2.**newSingleThreadExecutor方法**


```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

```
SingleThreadExecutor是单线程线程池，只有一个核心线程。
> 

> corePoolSize=1；核心线程池的数量为1
> maximumPoolSize=1;线程池最大的数量为1
> keepAliveTime =0L
> unit=毫秒
> workQueue=LinkedBlockingQueue


<br />当一个任务提交时，首先会创建一个核心线程来执行任务，如果超过核心线程的数量，将会放入队列中，因为LinkedBlockingQueue是长度Integer.MAX_VALUE的队列，可以认为是无界队列，因此往队列中可以插入无限多的任务，在资源有限的时候容易OOM异常，同时因为无界队列，maximumPoolSize和keepAliveTime参数将无效，压根就不会创建非核心线程。

<a name="ScSpP"></a>
#### 3.newFixedThreadPool方法


```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```


FixedThreadPool是固定核心线程的线程池，固定核心线程数由用户传入
> 

> corePoolSize=1；核心线程数用户自定义
> maximumPoolSize=1;线程池最大的数量用户自定义
> keepAliveTime =0L
> unit=毫秒
> workQueue=LinkedBlockingQueue
> 它和SingleThreadExcutor类似，唯一的区别就是核心线程数不同，并且由于使用的是LinkedBlockingQueue，在资源有限的时候容易引起OOM异常。
> <br />

<a name="DsG9Y"></a>
### 三.线程池线程的状态
线程池源码中线程状态
```java
 private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```


- **RUNNING** 
  - 状态说明：自然是运行状态，指可以接受任务执行队列里的任务线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0
  - 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0!

 private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

- **SHTUDOWN**
  - 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。 
  - 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。



- **STOP**
  - 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
  -  状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。
- **TIDYING**
  - 状态说明：当所有的任务已终止，任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
  - 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。



- **TERMINATED**
  -  状态说明：，当执行 terminated() 后会更新为这个状态，就变成TERMINATED状态。
  - 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。


<br />excute方法源码：
```java
 public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();;//获取当前线程池的状态 
        if (workerCountOf(c) < corePoolSize) {//当前线程数量小于 coreSize 时创建一个新的线程运行
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {//如果当前线程处于运行状态，并且写入阻塞队列成功
            int recheck = ctl.get();
            //双重检查，再次获取线程状态；如果线程状态变了（非运行状态）就需要从阻塞队列移除任务，并尝试判断线程是否全部执行完毕。同时执行拒绝策略。
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0) //如果当前线程池为空就新创建一个线程并执行。
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            //如果在第三步的判断为非运行状态，尝试新建线程，如果失败则执行拒绝策略
            reject(command);
    }

```


<a name="GfmpW"></a>
### 四.如何定义线程池参数


- **CPU 密集型任务(N+1)：** 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。



- **I/O 密集型任务(2N)：** 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。



- **阻塞队列:** 推荐使用有界队列，有界队列有助于避免资源耗尽的情况发生



- **拒绝策略 : **默认采用的是AbortPolicy拒绝策略，直接在程序中抛出RejectedExecutionException异常【因为是运行时异常，不强制catch】，这种处理方式不够优雅。处理拒绝策略有以下几种比较推荐：
  - 在程序中捕获RejectedExecutionException异常，在捕获异常中对任务进行处理。针对默认拒绝策略
  - 使用CallerRunsPolicy拒绝策略，该策略会将任务交给调用execute的线程执行【一般为主线程】，此时主线程将在一段时间内不能提交任何任务，从而使工作线程处理正在执行的任务。此时提交的线程将被保存在TCP队列中，TCP队列满将会影响客户端，这是一种平缓的性能降低
  - 自定义拒绝策略，只需要实现RejectedExecutionHandler接口即可
  - 如果任务不是特别重要，使用DiscardPolicy和DiscardOldestPolicy拒绝策略将任务丢弃也是可以的



<a name="gs7I2"></a>
### 五.优雅的关闭线程池

<br />有运行任务自然也有关闭任务，从上文提到的5个状态就能看出如何来关闭线程池。<br />
<br />其实无非就是两个方法shutdown()  , shutdownNow();<br />
<br />但是它们有重要的区别：

  - shutdown()执行后停止接收新任务，会把队列的任务执行完毕。
  - shutdownNow（），也是停止接收新任务，但会中断所有的任务，将线程池状态变为stop。
