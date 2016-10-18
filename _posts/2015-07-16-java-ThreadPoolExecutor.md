---
layout: post
title:  ThreadPoolExecutor简介
description: 
keywords: Java,Thread
category: Java
tags: [Java,Thread]
---

Java提供了Executors工厂方法来创建线程池，常见的方式有：

* Executors.newCachedThreadPool()（无界线程池，可以进行自动线程回收）

* Executors.newFixedThreadPool(int)（固定大小线程池）

* Executors.newSingleThreadExecutor()（单个后台线程）

* Executors.newScheduledThreadPool(int)（可供调度的固定线程池）

它们均为大多数使用场景预定义了设置，除此之外，还有手动配置线程池类`java.util.concurrent.ThreadPoolExecutor`。

<!-- more -->

####常用构造方法

{% highlight java %}
public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue,  
                          RejectedExecutionHandler handler)  
{% endhighlight %}

* corePoolSize： 线程池维护线程的最少数量 

* maximumPoolSize：线程池维护线程的最大数量 

* keepAliveTime： 线程池维护线程所允许的空闲时间 

* unit： 线程池维护线程所允许的空闲时间的单位 

* workQueue： 线程池所使用的缓冲队列 

* handler： 线程池对拒绝任务的处理策略 


####创建新线程

如果没有另外说明，则在同一个 ThreadGroup中一律使用 Executors.defaultThreadFactory() 创建线程，并且这些线程具有相同的 NORM_PRIORITY 优先级和非守护进程状态。通过提供不同的 ThreadFactory，可以改变线程的名称、线程组、优先级、守护进程状态，等等。如果从 new Thread() 返回 null 时 ThreadFactory 未能创建线程，则执行程序将继续运行，但不能执行任何任务。

默认情况下，即使核心线程最初只是在新任务到达时才创建和启动的，也可以使用方法 prestartCoreThread() 或 prestartAllCoreThreads() 对其进行动态重写。

一个任务通过 execute(Runnable)方法被添加到线程池，任务就是一个 Runnable类型的对象，任务的执行方法就是 Runnable类型对象的run()方法。 


####线程池说明

当一个任务通过execute(Runnable)方法欲添加到线程池时： 

* 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。 

* 如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。 

* 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。 

* 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。 

也就是：处理任务的优先级为： 核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。 

当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。 

####TimeUnit

unit可选的参数为java.util.concurrent.TimeUnit中的几个静态属性： 

* NANOSECONDS

* MICROSECONDS

* MILLISECONDS

* SECONDS

####RejectedExecutionHandler

handler有四个选择： 

* ThreadPoolExecutor.AbortPolicy() 

遭到拒绝将抛出java.util.concurrent.RejectedExecutionException运行时异常 

* ThreadPoolExecutor.CallerRunsPolicy() 

重试添加当前的任务，他会自动重复调用execute()方法 

* ThreadPoolExecutor.DiscardOldestPolicy() 

抛弃旧的任务 

* ThreadPoolExecutor.DiscardPolicy() 

抛弃当前的任务 

####排队策略

排队有三种通用策略：

* 直接提交

工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

* 无界队列

使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

* 有界队列

当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。

####挂钩方法

此类提供 protected 可重写的 beforeExecute(java.lang.Thread, java.lang.Runnable) 和 afterExecute(java.lang.Runnable, java.lang.Throwable) 方法，这两种方法分别在执行每个任务之前和之后调用。它们可用于操纵执行环境；例如，重新初始化ThreadLocal、搜集统计信息或添加日志条目。此外，还可以重写方法 terminated() 来执行 Executor 完全终止后需要完成的所有特殊处理。
如果挂钩或回调方法抛出异常，则内部辅助线程将依次失败并突然终止。

 

####队列维护

方法 getQueue() 允许出于监控和调试目的而访问工作队列。强烈反对出于其他任何目的而使用此方法。remove(java.lang.Runnable) 和 purge() 这两种方法可用于在取消大量已排队任务时帮助进行存储回收。
