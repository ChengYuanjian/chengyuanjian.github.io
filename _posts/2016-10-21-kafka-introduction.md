---
layout: post
title:  Kafka要点解析
description:
keywords: kafka
category: Kafka
tags: [Kafka]
---

### 什么是Kafka

Kafka是LinkedIn用scala开发的，基于发布／订阅的分布式消息系统，以易水平扩展和高性能特征著称。

### Kafka常用术语

* Broker：Kafka集群由多个实例组成，这个实例称为broker
* Topic：所有发布到Kafka集群的消息都有一个主题，称为topic（不同topic的消息在物理上分开存储）
* Partition：物理概念，每个topic包含一个或多个partition（默认为1），每个partition对应一个文件夹，每条消息会顺序追加到文件尾部，消息在文件中的位置称为offset
* Producer：生产者，负责发布消息到Broker
* Consumer：消费者，负责获取消息（可以为consumer指定group name进行分组，同一个topic的一条消息只能被同一个group内的一个consumer消费，但可以被不同group的consumer同时消费）
* Zookeeper：分布式协调管理服务，Kafka通过zookeeper进行集群管理，选举leader

<!-- more -->

### Kafka要点

#### 关于partition

* 一个topic一般对应至少一个partition，所有消息均匀分布到不同的partition。可以通过配置文件`server.properties`中的`num.partitions`进行设置，也可以在创建topic的时候人为指定
* 所有消息都会被append到partition中，顺序写磁盘，效率极高（高于随机写内存），这是其高性能的重要保证
* Kafka中无论消息是否被消费，都不会删除消息（这跟传统的MQ不太一样），出于磁盘资源的考虑，也不会永久保留。它提供两种方式删除旧数据，一是根据时间，二是根据partiton文件大小。可以通过配置文件`server.properties`修改，如下：

```properties
# The minimum age of a log file to be eligible for deletion
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824
```
* 正常情况下，consumer每消费一条消息，offset会线性增加。我们可以改变offset的值来重复消费一些消息。offset由consumer来控制，所以broker无需标记哪些消息被消费，也不用保证同一个group只有一个consumer消费一条消息，就不存在锁机制，这也是Kafka高性能的保证之一。

#### 关于replication和leader election

* replication的数量默认为1，可以在`server.properties`中的`default.replication.factor`进行设置，也可以在创建topic时指定
* 每个partition都有唯一的leader，replication和leader election配合提供了自动failover机制。所有的读写由leader完成，follower需要和leader保持同步
* leader会监视ISR（in-sync replicas），如果一个follower宕机或者落后太多，leader会将它从该列表中移除
* 一条消息只有被ISR中的所有follower都从leader复制过去才被认为已提交（committed），才能被消费。这样就避免leader宕机从而导致消息丢失
* ISR中所有的follower都挂了，就无法保证数据不丢失，此时有两种方案：1.等待ISR中任何一个replica恢复，选举为leader；2.选择第一个恢复的replica（不一定存在ISR中）作为leader

#### 关于消息可靠性传输

消息可靠性一般分为三种：

1. `At least once`：消息不会丢，但可能会重复
2. `At most once`：消息可能会丢，但不会重复
3. `Exactly once`：既不会丢也不会重复

Kafka默认保证了`At least once`：

* 当producer向broker发送消息时，一旦消息被commit，因为relication的存在，消息不会丢
* 当consumer从broker读取消息后，可以选择commit，该操作会被在zookeeper中存下offset。下一次再读时，会从offset下一条开始读取。如未commit，则会读取offset位置的数据

当若想实现`Exactly once`，则需要配合持久化和offset来实现。
