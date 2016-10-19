---
layout: post
title:  Redis数据过期策略
description:
keywords: Redis
category: Redis
tags: [Redis]
---

### Redis中Key的过期时间

通过EXPIRE key seconds命令来设置数据的过期时间。返回1表明设置成功，返回0表明key不存在或者不能成功设置。在key上设置了过期时间后key将在指定的秒数后被自动删除。指定过期时间的key在Redis中是不稳定的。

* 当key被DEL命令删除或者被SET、GETSET命令重置后与之关联的过期时间会被自动清除
* 使用PERSIST可以清除过期时间
* 使用rename只能改变key，但无法清除过期时间

<!-- more -->

### Redis过期键删除策略

Redis key过期的方式有三种：

* 被动删除：当读/写一个已经过期的key时，会触发惰性删除策略，直接删除掉这个过期key
* 主动删除：由于惰性删除策略无法保证冷数据被及时删掉，所以Redis会定期主动淘汰一批已过期的key
* 当前已用内存超过maxmemory限定时，触发主动清理策略

#### 被动删除

当key被操作时（例如get），redis才会被动检查该key是否过期，如果过期则删除之并且返回NIL。这种方式显然会造成大量的内存空间浪费。

#### 主动删除

Redis中，有个常规操作serverCron，它主要执行以下操作：

* 更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等
* 清理数据库中的过期键值对
* 对不合理的数据库进行大小调整
* 关闭和清理连接失效的客户端
* 尝试进行AOF或RDB持久化操作
* 如果服务器是主节点的话，对附属节点进行定期同步
* 如果处于集群模式的话，对集群进行定期同步和连接测试

Redis将serverCron作为时间事件来循环运行，直到服务器关闭为止。默认情况下每秒运行10次，2.8版本之后，可以修改redis.conf中`hz`参数进行调整，一般建议不超过100。
除了`hz`主动淘汰的频率外，Redis对每次淘汰任务执行的最大时长也有一个限定timelimit，这样保证了每次主动淘汰不会过多阻塞应用请求。它们是倒数的关系，也就是说hz配置越大，timelimit就越小。

当hz为10时，Redis每秒做10次如下的步骤：

1. 随机测试100个设置了过期时间的key
2. 删除所有发现的已过期的key
3. 若删除的key超过25个则重复步骤1

需要注意的是，在主从模式下，只有master节点才会执行上述这两种过期删除策略，然后把删除操作`del key`同步到slave结点。

#### 超过maxmemory

当前已用内存超过maxmemory限定时，触发主动清理策略，主要包含以下几种（可通过`maxmemory-policy`指定）：

* volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
* volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
* volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
* allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
* allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
* no-enviction：禁止驱逐数据，注意这个清理过程是阻塞的，直到清理出足够的内存空间。所以如果在达到maxmemory并且调用方还在不断写入的情况下，可能会反复触发主动清理策略，导致请求会有一定的延迟。

清理时会根据配置的`maxmemory-policy`来做适当的清理（一般是LRU或TTL），并不是针对redis的所有key，而是以配置文件中的`maxmemory-samples`个key作为样本池进行抽样清理。
`maxmemory-samples`默认配置为5，如果增加（一般不要超过10），会提高LRU或TTL的精准度，但会消耗更多的CPU。

**尽量不要触发maxmemory，最好在mem_used内存占用达到maxmemory的一定比例后，需要考虑调大hz以加快淘汰，或者进行集群扩容。**
