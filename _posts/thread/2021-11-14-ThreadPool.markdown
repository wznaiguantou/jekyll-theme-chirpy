---
layout: post
title:  "线程池(一)"
date:   2021-11-14 10:29:40 +0800
categories: 线程学习
---

### 线程池的背景

一种**多线程**
的使用模式。随着计算机的发展，[摩尔定律](https://wiki.mbalib.com/wiki/%E6%91%A9%E5%B0%94%E5%AE%9A%E5%BE%8B)日渐失效，多核CPU成为主流。使用多线程并行计算成为开发人员提升服务器性能的基本武器。
JUC提供的线程池：**ThreadPoolExecutor.class** 帮助开发人员管理线程并方便的执行并行任务。

### 频繁创建线程的局限性

1. 线程的创建、销毁极具消耗系统资源，而线程是稀缺资源，目前Java线程和OS里的线程是1:1的关系。
2. 创建、销毁单个线程从OS的角度是需要消耗一定的时间的，频繁创建线程的时间：消耗时间 * NThread
3. 频繁创建线程，会导致系统调度的失衡，降低系统的稳定性，还容易引发OOM。
4. 从开发角度，线程的定时、延时等高级特性无法扩展。

### 线程池的引入

在当下并发的环境下,系统需要执行多少任务，投入多少资源。这些不确定因素都会导致系统的不稳定。 池化("Pooling")技术的引入， 解决的核心问题就是线程资源管理问题。

- 统一资源管理: 不同业务分配不同的线程池，避免了业务之间对线程的需求而产生的影响.e.g: 线程阻塞
- 线程可复用: 减少线程的创建、销毁带来的开销，性能更好。
- 监控: 对线程池的资源进行实时监控。
- 控制并发数: 有效控制系统最大并发线程数，提升系统资源的利用率，避免系统过多的资源竞争，从而避免阻塞。

### **线程池的状态和参数**

```java

    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    //Integer.Size = 32
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

```

#### 线程池的状态

高3位来描述线程池的状态。

| 状态         | Code        | 描述         |
|:------------|:------------|:------------|
| Running     | (-1 << 29) =>   |  运行中，线程池处于可以接收外部task的状态   |
| Shutdown    | (0 << 29) => 0  |  关闭线程池，但是还执行队列里的task|
| Stop        | (1 << 29) => 0  |  关闭线程池，放弃队列里的任务，正在执行的任务也添加中断标记位|
| Tidying     | (2 << 29) => 0  |  一个Stop到终结的过程，这个时候会触发 AfterExecute()这个钩子函数|
| Terminated  | (3 << 29) => 0  |  线程池完全终结。所以的workers都终结了。|

### ThreadPoolExecutor 源码解析

#### 1. 线程池定义

如表格所示, 如下7个参数是构建一个线程池所需要的定义。我们可以先看下这7个参数的定义

|线程池参数          | 描述         |
|:-----------------|:------------|
| corePoolSize     |  核心线程数，在队列未满的情况下，工作线程的数量就是<= corePoolSize   |
| maxPoolSize      |  最大线程数，在队列已满且还是有新的任务进来后，则会启动新的worker来消费队列里的任务，此时如果任务超过了最大线程数，就开始出发拒绝策略  |
| keepAliveTime    |  线程在没事情做的时候做大存活时间，底层代码会根据填写的时间和TimeUnit转化成毫秒   |
| timeUnit         |  时间单位，计算线程存活时间用的。
| queue            |  队列，LinkedBlockingQueue / DelayQueue,用来处理来不及消费的任务。FIFO   |
| threadFactory    |  线程工厂，负责创建线程。可以对创建的线程进行一些自定义，比如name, ExceptionHandler   |
| policy           |  rejectPolicy,出现队列已满，且超出最大线程线程数的时候出现，已经超出线程池的临界点，实现中有一种背压方式的实现，CallerRunsPolicy   |

```java






```

#### 2. 线程池的常用方法

#### 2.1. 线程池的常用方法

基于扩展线程池进行扩展,此处给了2个钩子函数的示例.

```java

 /** 示例扩展线程池的方式 */
    static class MyThreadPoolExecutors extends ThreadPoolExecutor {

        public MyThreadPoolExecutors(
                int corePoolSize,
                int maximumPoolSize,
                long keepAliveTime,
                TimeUnit unit,
                BlockingQueue<Runnable> workQueue) {
            super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
        }

        /**
         * 如果我在线程内部没有异常处理机制，那我在定义线程池的时候可以给予补偿
         *
         * @param r runnable task
         * @param t throwable
         */
        @Override
        protected void afterExecute(Runnable r, Throwable t) {}


        @Override
        protected void beforeExecute(Runnable r, Throwable t) {}

    }
```

其他常用的一些方法

```java
        /* 预启动核心线程 */
        EXECUTOR.ensurePrestart();

        /* 允许核心线程超时 */
        EXECUTOR.allowsCoreThreadTimeOut();

        /*关闭线程池的方法*/
        EXECUTOR.shutdown();
        /* 立即关闭线程池*/
        EXECUTOR.shutdownNow();

        // 如果队列里的任务被取消了，且这些被取消的任务还没有被工作线程消费到。那就可以被遍历所有的任务后删除
        EXECUTOR.purge();

        /**
         * submit method and return {@link Future<V>}
         *
         * @see ThreadPoolExecutor#submit(Runnable)
         * @see ThreadPoolExecutor#submit(Callable)
         * @see ThreadPoolExecutor#submit(Callable)
         */
        var future = EXECUTOR.submit(() -> System.out.println("1"));

        /**
         * execute runnable method
         *
         * @see ThreadPoolExecutor#execute(Runnable)
         * @return void
         */
        EXECUTOR.execute(() -> System.out.println("1"));
```

### ScheduledThreadPoolExecutor
```java
package com.example.demo.thread.analysis.scheduled;

import lombok.extern.slf4j.Slf4j;
import lombok.var;
import org.junit.jupiter.api.Test;

import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @author zhijun.gao
 * @since 2021/11/22 23:06
 */
@Slf4j
public class ScheduledPoolTest {

    /** run command once with delay time */
    @Test
    void schedule() {

        var scheduledPool = new ScheduledThreadPoolExecutor(1);
        log.info("start");
        scheduledPool.schedule(() -> log.info("1"), 2, TimeUnit.SECONDS);
        foreach:
        for (; ; ) {

            System.out.println("1");
            for (; ; ) {
                try {
                    TimeUnit.SECONDS.sleep(3);
                    log.info("break");
                    break foreach;
                } catch (InterruptedException e) {
                    log.info("", e);
                }
            }
        }
        log.info("end");
    }

    /**
     * scheduled commands one by one
     *
     * <p>result
     * <li>00:10:19.375 [main] INFO ScheduledPoolTest - start
     * <li>00:10:19.393 [pool-1-thread-1] INFO ScheduledPoolTest - executing command
     * <li>00:10:21.396 [pool-1-thread-1] INFO ScheduledPoolTest - executing command
     * <li>00:10:23.397 [pool-1-thread-2] INFO ScheduledPoolTest - executing command
     */
    @Test
    void scheduleOneByOne() {

        var scheduledPool = new ScheduledThreadPoolExecutor(3);
        log.info("start");
        scheduledPool.scheduleAtFixedRate(
                () -> log.info("executing command"), 0, 2, TimeUnit.SECONDS);
        for (; ; ) {}
    }

    /**
     * scheduled by delay time. if executing time > delayTime, execute command immediately
     *
     * <p>result
     * <li>00:01:40.532 [main] INFO ScheduledPoolTest - start
     * <li>00:01:43.534 [pool-1-thread-1] INFO ScheduledPoolTest - break
     * <li>00:01:46.538 [pool-1-thread-1] INFO ScheduledPoolTest - break
     * <li>00:01:49.543 [pool-1-thread-2] INFO ScheduledPoolTest - break
     * <li>00:01:52.544 [pool-1-thread-2] INFO ScheduledPoolTest - break
     */
    @Test
    void scheduleAtFixedRate() {
        var scheduledPool = new ScheduledThreadPoolExecutor(3);
        log.info("start");
        scheduledPool.scheduleAtFixedRate(
                () -> {
                    try {
                        TimeUnit.SECONDS.sleep(3);
                        log.info("break");
                    } catch (InterruptedException e) {
                        log.info("", e);
                    }
                },
                0,
                2,
                TimeUnit.SECONDS);
        for (; ; ) {}
    }

    /**
     * executing time of command + delay time
     *
     * <p>result
     * <li>23:57:39.668 [main] INFO ScheduledPoolTest - start
     * <li>23:57:42.671 [pool-1-thread-1] INFO ScheduledPoolTest - break
     * <li>23:57:47.674 [pool-1-thread-1] INFO ScheduledPoolTest - break
     * <li>23:57:52.680 [pool-1-thread-1] INFO ScheduledPoolTest - break
     */
    @Test
    void scheduleWithFixedDelay() {
        var scheduledPool = new ScheduledThreadPoolExecutor(1);
        log.info("start");
        scheduledPool.scheduleWithFixedDelay(
                () -> {
                    try {
                        TimeUnit.SECONDS.sleep(3);
                        log.info("break");
                    } catch (InterruptedException e) {
                        log.info("", e);
                    }
                },
                0,
                2,
                TimeUnit.SECONDS);
        for (; ; ) {}
    }
}
```

### Tips:

- Q1. 假设CPU是单核的，那么还有必要设置多线程么?
  <br>
  A1: 看场景，

1. 如果是一直占用CPU进行计算的任务，那么多线程是毫无意义的，还浪费CPU的线程上下文切换，损耗性能.
   <br>

```java
 void run(){
   while(true){
    i++;
  }
 }
```

2. 如果是不是那种CPU密集型计算的任务，那么代表任务是I/O型的，自然可能会在阻塞期间不占用CPU的资源，那么这个时候 多线程的优势就体现出来了。

- Q2. 线程池execute方法中的**FirstTask**是什么作用？
  <br>
  A2: 用来表示不是队列里获取的任务，是该线程第一次启动后需要执行的任务。

- Q3. 线程池线程数一般多少比较合适？
  <br>
  A1:

- Q4. 线程池的队列多少合适?
  <br>
  A2:
  <br>
- Q5.

