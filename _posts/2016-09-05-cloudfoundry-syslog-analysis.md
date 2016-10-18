---
layout: post
title:  CloudFoundry日志分析——firehose＋rsyslog＋kafka＋storm
description: 
keywords: CloudFoundry,PaaS,kafka,storm
category: CloudFoundry
tags: [CloudFoundry,PaaS]
---


* 操作系统：服务端Ubuntu 14.04／客户端Mac OS X 10.11.6
* JDK：JDK1.8.0
* GO：GO1.7


----

## firehose-to-syslog

### 什么是firehose-to-syslog 

Firehose是基于websocket的，用来收集事件数据，包括日志、http事件、应用和容器的度量数据等（注意Cloud Foundry系统组件本身的日志是不被包含的）。

firehose-to-syslog是官方提供用来把firehose获取的信息推向rsyslog的工具：[GitHub地址](https://github.com/cloudfoundry-community/firehose-to-syslog)


<!-- more -->


### 配置firehose-to-syslog

firehose-to-syslog使用go语言编写，所以安装go语言运行环境并配置环境变量。

* [点击下载go安装包](https://golang.org/dl/)
* 配置环境变量：

```sh
export GOROOT=/usr/local/go
export PATH=/usr/local/go/bin:$PATH
export GOPATH=/Users/CYJ/go
```

* 查看版本：`go version`查看环境变量：`go env`
* 下载工程：`go get github.com/cloudfoundry-community/firehose-to-syslog`
* 进入目录：`cd $GOPATH/src/github.com/cloudfoundry-community/firehose-to-syslog`
* 编译：`go build`，此时会生成一个可执行的命令：`firehose-to-syslog`，可以通过加`-o`选项制定其他名字
* 执行命令进行推送：

```./firehose-to-syslog --api-endpoint="https://api.truepaas.cn"  --syslog-server=192.168.199.236:514 --syslog-protocol="udp" --skip-ssl-validation --debug --doppler-endpoint="wss://doppler.truepaas.cn:4443" --user="admin" --password="admin" --events="HttpStartStop"
```

----------

## Kafka

### 什么是Kafka

Kafka是一种高吞吐量的分布式发布订阅消息框架，最初由LinkedIn公司开发，之后成为Apache项目的一部分。

### 安装Kafka

1. [点击下载安装包](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.0.0/kafka_2.11-0.10.0.0.tgz)
2. 解压：`tar -xzf kafka_2.11-0.10.0.0.tgz`
3. 顺序执行以下命令启动：

 * `cd kafka_2.11-0.10.0.0`
 * `bin/zookeeper-server-start.sh config/zookeeper.properties`
 * `bin/kafka-server-start.sh config/server.properties`

以上便完成了单节点kafka的启动。

__Kafka依赖JVM，如果未装JDK，则需执行下载：__

```wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz"
```

__然后解压，配置环境变量（这里不赘述）方可正常运行。__

-------

### Kafaka示例（单节点）

1. 创建主题`topic`——cyj：
`bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic cyj`
2. 创建生产者`producer`：
`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic cyj`
3. 另起终端，创建消费者`consumer`：
`bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic cyj --from-beginning`
4. 此时可以通过生产者发送消息，而消费者会自动接收消息，使用`Ctrl＋C`退出会话

### Kafaka示例（集群）
以上是创建个单个broker，在实际场景中，我们可能需要多个borker（以两个节点为例）:

1.复制一份配置文件：`cp config/server.properties config/server-1.properties`
2.修改配置：`vi config/server-1.properties`

```sh
broker.id=1 #broker的id，集群里必须唯一，一般从0开始
listeners=PLAINTEXT://:9093 #主机端口配置
log.dir=/tmp/kafka-logs-1 #日志路径
```
3.分别启动两个broker：
`bin/kafka-server-start.sh config/server.properties`
`bin/kafka-server-start.sh config/server-1.properties`

4.查看当前节点信息：`bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic`

部分信息说明：

* leader：负责处理消息的节点，leader是从所有节点中随机选择的
* replicas：列出了所有的节点，不管节点是否在服务中
* isr：是正在服务中的节点

5.创建主题：
`bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 1 --topic my-replicated-topic`
6.同上节2、3步创建生产者和消费者后，可以正常收发消息。当我们人为kill掉某个节点时：`ps | grep server-1.properties & kill -9 pid`，消息发送不会受任何影响，由此可见，集群生效。


[官方文档](http://kafka.apache.org/documentation.html#quickstart)

-------

## Rsyslog

### 什么是Rsyslog

Rsyslog是一个自由软件，目标是提供一个更可靠的系统日志守护进程和配置，可以看作增强版的syslog。

### 安装Rsyslog

1. 选择资源库：`add-apt-repository ppa:adiscon/v8-stable`
2. 更新apt缓存：`apt-get update`
3. 安装：`apt-get install rsyslog`
4. 查看当前版本：`rsyslogd -version`
5. 安装kafka插件：`apt-get install rsyslog-kafka rsyslog-imptcp`
6. 安装librdkafka依赖：

```sh
git clone https://github.com/edenhill/librdkafka.git
cd librdkafka
./configure --prefix=/usr
make
sudo make install
```
安装成功后 `/usr/lib/rsyslog` 目录下会出现相关的so库，rsyslog会从这个目录下去加载相关的动态库模块。

__由于rsyslog在8.7.0以后的版本才支持kafka，所以在第一步选择安装v8的稳定版；通过第四步可以看到当前版本号为8.21.0__

----

### 配置Rsyslog

默认rsyslog的配置文件是`/etc/rsyslog.conf` 和 `/etc/rsyslog.d`下的配置。
最好不要轻易修改全局的`rsyslog.conf`, 我们在`/etc/rsyslog.d`目录下新建一个`cyj.conf`文件，配置如下：

```bash

module(load="imudp") 
input(type="imudp" port="514")

module(load="imptcp") 
input(type="imptcp" port="514")

template(name="jtpl"
         type="string"
         string="%hostname%<-+>%syslogtag%<-+>%msg%\n"
        ) 

module(load="omkafka")

if $inputname == "imudp" or $inputname == "imtcp" then {
    action (type="omkafka" topic="my-replicated-topic" broker="localhost" partitions.auto="on" template="jtpl" confParam=["compression.codec=snappy", "socket.keepalive.enable=true"])
}

```

__完整模板格式：__`%property:fromChar:toChar:options%`
从左至右依次表示属性、开始字符序号、结束字符序号、格式选项。

输入`service rsyslog restart`重新启动rsyslog使得配置生效。

验证步骤：

* 输出任意日志：`logger "hello，world"`
* 查看日志是否生成：`tail /var/log/syslog`
* 使用kafka客户端命令提取目标主题报文：`kafkacat -C -b 192.168.199.236:9092 -t my-replicated-topic`或者创建一个消费者consumer连接目标主题获取报文。


[官方文档](http://www.rsyslog.com/guides-for-rsyslog/)

接下来，我们就可以用Storm的Topology作为Consumer，来订阅对应的主题，获取日志做实时分析处理。

----

## Storm整合Kafka

Storm提供了Kafka的官方支持，使用Java编写：[GitHub地址]
(https://github.com/apache/storm/tree/master/external)

数据流程：

1. Kafka Producer生成topic1主题的消息　
2. Storm中有个Topology，包含了KafkaSpout、SenqueceBolt、KafkaBolt三个组件。其中KafkaSpout订阅了topic1主题消息，然后发送给SenqueceBolt加工处理，最后数据由KafkaBolt生成topic2主题消息反馈给Kafka（如需要）
3. Kafka Consumer负责消费topic2主题的消息（如需要）

__部分关键代码示例：__

```java
// 配置Zookeeper地址
BrokerHosts brokerHosts = new ZkHosts("node1:2181,node2:2181,node3:2181");
// 配置Kafka订阅的Topic，以及zookeeper中数据节点目录和名字
SpoutConfig spoutConfig = new SpoutConfig(brokerHosts, "topic1", "/zkkafkaspout" , "kafkaspout");

TopologyBuilder builder = new TopologyBuilder();   
builder.setSpout("spout", new KafkaSpout(spoutConfig));  
builder.setBolt("bolt", new SenqueceBolt()).shuffleGrouping("spout"); 
builder.setBolt("kafkabolt", new KafkaBolt<String, Integer>()).shuffleGrouping("bolt"); 
```
更多详情可参考[官方Demo](https://github.com/apache/storm/tree/master/external/storm-kafka)

