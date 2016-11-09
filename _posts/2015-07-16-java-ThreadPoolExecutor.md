---
layout: post
title:  java.util.concurrent之ThreadPoolExecutor简介
description:
keywords: Java,Thread
category: Java
tags: [Java,Thread,concurrent]
---

Java从1.5开始提供了`java.util.concurrent`包，可以让并发编程变得更为简单。本篇先讲一下线程池。
线程池的顶级接口是`Executor`，仅定义了一个`execute`方法，所以严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`ExecutorService`（继承自Executor），提供了丰富的方法用做线程池管理。
接下来是实现了`ExecutorService`接口的抽象类`AbstractExecutorService`，再到我们经常使用的实现类`ThreadPoolExecutor`

<!-- more -->

#### 常用构造方法

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

大多数情况下，JDK建议我们使用Executors工厂方法来创建线程池，常见的方式有：

* Executors.newSingleThreadExecutor()（单个后台线程）

* Executors.newFixedThreadPool(int)（固定大小线程池）

* Executors.newCachedThreadPool()（无界线程池，可以进行自动线程回收）

* Executors.newScheduledThreadPool(int)（可供调度的固定线程池）

其中，前三个方法是调用`ThreadPoolExecutor`的构造方法进行封装，第四个是调用`ScheduledThreadPoolExecutor`的构造方法进行封装。

__ScheduledThreadPoolExecutor是ThreadPoolExecutor的子类，同时实现了ScheduledExecutorService接口（该接口是ExecutorService的子接口）__

#### 线程池说明

当一个任务通过execute(Runnable)方法欲添加到线程池时：

* 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。

* 如果此时线程池中的数量等于corePoolSize，但是缓冲队列workQueue未满，那么任务被放入缓冲队列。

* 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。

* 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。

也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

当线程池中的线程数量大于corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

#### TimeUnit

unit可选的参数为java.util.concurrent.TimeUnit中的几个静态属性：

* NANOSECONDS

* MICROSECONDS

* MILLISECONDS

* SECONDS

#### RejectedExecutionHandler

handler有四个选择：

* ThreadPoolExecutor.AbortPolicy()

遭到拒绝将抛出java.util.concurrent.RejectedExecutionException运行时异常

* ThreadPoolExecutor.CallerRunsPolicy()

重试添加当前的任务，他会自动重复调用execute()方法

* ThreadPoolExecutor.DiscardOldestPolicy()

抛弃旧的任务

* ThreadPoolExecutor.DiscardPolicy()

抛弃当前的任务

#### 排队策略

排队有三种通用策略：

* 直接提交

工作队列的默认选项是SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。newCachedThreadPool采用该策略。

* 无界队列

使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。newSingleThreadExecutor和newFixedThreadPool采用该策略。

* 有界队列

当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。JDK不推荐使用。

