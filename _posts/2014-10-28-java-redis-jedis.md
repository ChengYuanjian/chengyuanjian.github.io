---
layout: post
title:  Redis基础 – Jave客户端Jedis
description: 
keywords: Java,Redis,Jedis
category: Java
tags: [Java,Redis,Jedis]
---

####什么是Redis

Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库。通常用来存储结构化的数据，和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。

####Redis客户端

针对java语言，redis client也提供了多种客户端支持，按照推荐类型依次是：Jedis、Redisson、JRedis、JDBC-Redis、RJC、redis-protocol、aredis、lettuce。参见[官方文档](http://redis.io/clients)，本文将使用Jedis访问Redis。

####准备工作

* [Redis下载](https://github.com/MSOpenTech/redis)，当前版本为2.8

* [Jedis下载](https://github.com/xetorthio/jedis)

* [Redis命令目录](http://www.redisdoc.com/en/latest/)

<!-- more -->

####Redis Server安装

#####1.Windows

非常简单，下载zip包后解压，添加至环境变量。复制出redis.conf配置文件至安装目录后，即可使用。一般会有以下几个命令供调用：

* redis-benchmark：性能测试，用以模拟同时由N个客户端发送M个 SETs/GETs 查询 (类似于 Apache 的ab 工具).

* redis-check-aof：更新日志检查

* redis-check-dump：本地数据库检查

* redis-cli：客户端

* redis-server：服务端　

#####2.Linux

稍微复杂一些，具体可以参见[这里](http://www.cnblogs.com/zhuhongbao/archive/2013/06/04/3117997.html)

####Redis配置文件

redis.conf不做更改也可以使用(默认端口为6379)。详细的参数配置说明，可以参见[这里](http://www.cnblogs.com/wenanry/archive/2012/02/26/2368398.html)

####Jedis访问Redis

#####1.必需jar包

导入jedis-2.1.0.jar和commons-pool-1.5.4.jar

#####2.构造非切片连接池JedisPool和切片连接池ShardedJedisPool

{% highlight java %}
JedisPoolConfig config = new JedisPoolConfig();
//控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取；
//如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
config.setMaxActive(500);
//控制一个pool最多有多少个状态为idle(空闲的)的jedis实例。
config.setMaxIdle(5);
//表示当borrow(引入)一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛出JedisConnectionException；
config.setMaxWait(1000 * 100);
//在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的；
config.setTestOnBorrow(true);
JedisPool pool = new JedisPool(config, "192.168.2.191", 8888);//非切片池

List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>(); 
shards.add(new JedisShardInfo("127.0.0.1", 6379, "master")); 
ShardedJedisPool shardedJedisPool = new ShardedJedisPool(config, shards)//切片池
{% endhighlight %}

#####3.非切片连接Jedis和切片连接ShardedJedis 

{% highlight java %}
Jedis jedis = pool.getResource();//从连接池获取Jedis实例
ShardedJedis shardedjedis = shardedJedisPool.getResource();
{% endhighlight %}

#####4.常用操作

* set(k,v)：增加k-v对，存在则覆盖

* get(k)：获取k对应的v

* append(k,v)：追加v至k对应值之后

* del(k)：删除k-v对

* mset(k1,v1,k2,v2)：批量增加k-v对

* exists(k)：判断是否存在k

* getSet(k,v)：获取并更改

* 更多可参见[API](http://tool.oschina.net/uploads/apidocs/)

#####5.返回资源

{% highlight java %}
//释放redis对象
pool.returnBrokenResource(jedis);
{% endhighlight %}

####参考资料

* [java对redis的基本操作](http://www.cnblogs.com/edisonfeng/p/3571870.html)

* [Redis的Java客户端Jedis的八种调用方式](http://www.blogways.net/blog/2013/06/02/jedis-demo.html)
