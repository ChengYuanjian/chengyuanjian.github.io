---
layout: post
title:  CloudFoundry架构组成
description: 
keywords: CloudFoundry,PaaS
category: CloudFoundry
tags: [CloudFoundry,PaaS,Architecture]
---

### Cloud Foundry简介

Cloud Foundry是VMware推出的业界第一个开源PaaS云平台，它支持多种框架、语言、运行时环境、云平台及应用服务。
它有三个显著的特点：

* Cloud Foundry支持各种框架，包括Spring for Java，.NET，Ruby on Rails，Node.js，Grails，Scala on Lift以及更多合作伙伴提供的框架（如Python，PHP等)。
* Cloud Foundry将应用和应用依赖的服务相分开，采用应用和服务绑定的机制。这些应用服务包括PostgreSQL，MySQL，SQL Server，MongoDB，Redis以及第三方和开源社区的应用服务。
* Cloud Foundry可以灵活的部署在公有云、私有云或者混合云之上，如vSphere/vCloud，AWS，OpenStack，Rackspace等多种云环境中，当然也支持物理机。

<!-- more -->

----

### Cloud Foundry核心组件

Cloud Foundry是由多个相对独立的模块构成的分布式系统，每个模块单独存在和运行，各模块之间通过消息机制进行通信。Cloud Foundry各模块本身是基于Ruby语言开发的，每个部分可以认为拿来即可运行，不存在编译等过程。

#### Router

Router在Cloud Foundry中是对所有进来的请求进行路由。进入Router的请求主要有两类。

* 第一类是管理请求，来自VMC Client或者STS的，由Cloud Foundry使用者发出。这类请求会被路由到Cloud Controller组件处理。

* 第二类是对所部署的App的访问请求。这部分请求会被路由到DEA组件中。

Router本身是无状态，可扩展。实际生产中，会部署多个Router做集群，通过nginx、DNS等方式来做负载均衡。

#### Authentication

包含两个组件，一个是Login Server，负责登录认证，一个是OAuth2 Server(UAA，User Account and Authentication)，负责身份管理。

#### Cloud Controller

即与VMC（Cloud Controller命令行界面）和STS（Eclipse插件）交互的服务器端，它收到指令后发消息到各模块，管理整个云的运行，相当于Cloud Foundry的大脑。对外只开放Rest接口。

#### DEA (Droplet Execution Agent)

首先要解释下什么叫做Droplet。Droplet指把提交的源代码及Cloud Foundry配置好的运行环境(如Java Web就是一个Tomcat)，再加一些控制脚本，如start/stop等，全部打包在一起的文件。这个过程是由Cloud Controller配合Stager完成的。Cloud Foundry会保存这个Droplet至blobstore，直到启动(start)一个App时，一台部署了DEA模块的服务器会去blobstore下载Droplet的副本去运行。
如果将App扩展到10个实例(instance)，那么这个Droplet就会被复制10份，供10台DEA服务器运行。
DEA由三部分组成，agent、warden和NAT网关。

* agent对外发布和订阅NATS对应的主题消息，对内根据消息操作对应的应用。
* warden是Cloud Foundry中的虚拟化容器，进行资源隔离。每个应用被一个warden包裹，并分配一个虚拟网卡。多个warden组成一个本地子网，这样Cloud Foundry可以根据网卡进行网络资源分配。

#### Health Manager

它用于监控DEA上各个应用的运行信息，包括状态、心跳、负载等。然后进行统计分析、报告、发出告警等。

#### Services

Cloud Foundry把Service模块设计成一个独立的、插件式的模块，便于第三方方便地把自己的服务整合成Cloud Foundry服务。
Services包含Service Gateway和Service Node两部分。

* Gateway用于接收Cloud Controller的创建、删除、绑定、解绑的指令，并下发到Service Node中。
* Service Node是真正运行服务本身的节点。

现实情况中，种种原因使有些系统服务难以或不愿意迁移到云端，为此Cloud Foundry 引入了Service Broker模块。可以使部署在Cloud Foundry上的应用能访问云外的服务。

#### NATS (Message bus)

Cloud Foundry的架构是基于消息发布和订阅的。联系各模块的是一个叫NATS的组件。NATS是由Cloud Foundry开发的一个基于事件驱动的、轻量级的消息系统。它基于EventMachine实现。各模块会根据当前的行为，向对应主题发布消息，同时也按照需要监听多个主题，彼此以消息进行通信。

#### Logging and Statistics

包含Metrics Collector和Log Aggregator。前者用来采集监控数据，后者用来收集应用日志。


