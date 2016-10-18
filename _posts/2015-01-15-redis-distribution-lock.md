---
layout: post
title:  Quartz利用Redis分布式锁实现Master-Slave模式
description: 
keywords: Java,Redis,Jedis,Quartz
category: Java
tags: [Java,Redis,Jedis,Quartz]
---

####背景

Spring+Quartz可以很方便地实现任务调度，也支持cluster模式，参见[spring整合quartz并持久化](http://haiziwoainixx.iteye.com/blog/1838055)。
但需要在数据库创建12张表。杀鸡焉用牛刀？对于简单的分布式场景，有没有更好的方式呢？答案是确定的。

####解决方案

利用Redis分布式锁，判断当前任务是否获得锁，如果获得则为master执行任务，否则就是slave候命。

<!-- more -->

####具体步骤

* 1.设置时间戳

{% highlight java %}
//系统时间+超时时间+1
long timestamp = System.currentTimeMillis() + defaulttimeout + 1;
{% endhighlight %}

* 2.判断是否获取锁

{% highlight java %}
//1表示获取锁，0表示未获取，多台server仅有一个会获取锁
long lock = jedis.setnx(lockstamp, timestamp + "");
{% endhighlight %}

* 3.标记master/slave

{% highlight java %}
//如果自己获取锁，或者原有的master已经超时，则标记自己为master，同时改变锁的超时时间
if (lock == 1 || (System.currentTimeMillis() > Long.parseLong(jedis
							.get(lockstamp)) && System.currentTimeMillis() > Long
							.parseLong(jedis.getSet(lockstamp, timestamp + "")))) {
	logger.info("I'm master, I will do some job.");
	break;
} else {//否则，标记自己为slave，并等待
	try {
		logger.info("I'm slave, I need to stand by.");
		Thread.sleep(10);
		} catch (InterruptedException e) {
		logger.error("Failed to release lock", e);
		}
	}
{% endhighlight %}

* 4.释放锁

{% highlight java %}
//如果未超时，则释放锁
if (System.currentTimeMillis() < Long.parseLong(jedis.get(lockstamp))) {
			jedis.del(lockstamp);
		}
{% endhighlight %}

####完整代码

{% highlight java %}
public void execute() {
		long lock = 0;
		while (lock != 1) {
			long timestamp = System.currentTimeMillis() + defaulttimeout + 1;

			lock = jedis.setnx(lockstamp, timestamp + "");

			if (lock == 1
					|| (System.currentTimeMillis() > Long.parseLong(jedis
							.get(lockstamp)) && System.currentTimeMillis() > Long
							.parseLong(jedis.getSet(lockstamp, timestamp + "")))) {
				logger.info("I'm master, I will do some job.");
				break;
			} else {
				try {
					logger.info("I'm slave, I need to stand by.");
					Thread.sleep(10);
				} catch (InterruptedException e) {
					logger.error("Failed to release lock", e);
				}
			}

		}
		logger.info("=========Job start=========");
    //TODO do something
		logger.info("=========Job end=========");

		if (System.currentTimeMillis() < Long.parseLong(jedis.get(lockstamp))) {
			jedis.del(lockstamp);
		}
	}
{% endhighlight %}

####相关文章

[Redis基础 – Jave客户端Jedis](http://chengyuanjian.github.io/java/2014-10/java-redis-jedis.html)
