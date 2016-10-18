---
layout: post
title:  REST简述
description: 
keywords: REST
category: Other
tags: [REST]
---

大概2010年的时候，听说了REST这个单词，但一直没去关注和了解。直到今年年初换工作的时候，有道笔试题（顺便吐槽一下社招还有笔试的奇葩公司）正好是关于Spring是如何实现REST的，我顿时愕然，不知该如何回答合适。虽然这道题并没有影响最终的笔试结果，但一直让我耿耿于怀至今。新的工作并没有任何使用REST的场景，也没有往REST发展的迹象，我很难理解当初为什么会出现这道题目。利用工余时间Google了相关资料，也研究了一下REST相关代码实现，对这个知识点做一个简单的梳理。


###REST是什么？

REST是英文Representational State Transfer的缩写，中文翻译：表述性状态转移/表现层状态转化。是Roy Fielding博士在2000年他的博士论文中提出来的一种软件架构风格。它是一种针对网络应用的设计和开发方式，可以降低开发的复杂性，提高系统的可伸缩性。满足这些约束条件和原则的应用程序或设计就是RESTful。

<!-- more -->

###REST基本原则

####一：所有事物都被抽象为资源（resource）

REST的名称省略了主语，指的是"资源"（Resources）的"表现层"。REST中的资源所指的不是数据，而是数据和表现形式的组合。表现形式在HTTP请求的头信息中用Accept和Content-Type字段指定。REST架构是针对Web应用而设计的，每种资源对应一个特定的URL。用__目录结构风格的URL__设计来表示资源，可以使客户端更容易理解和操作资源。
如[api.github.com](https://api.github.com/)的设计：

{% highlight json %}
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  "organization_repositories_url": "https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}"
}
{% endhighlight %}

-----------------------------

####二：使用HTTP的方法进行资源访问（CRUD）

客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

* 1.使用POST创建资源（C）

* 2.使用GET读取资源（R）

* 3.使用PUT更新资源（U）

* 4.使用DELETE删除资源（D）

由于资源和URI是一一对应的，执行这些操作的时候URI是没有变化的，这和以往的 Web开发有很大的区别。正由于这一点，极大的简化了Web开发，也使得URI可以被设计成更为直观的反映资源的结构，这种URI的设计被称作RESTful的URI。

-------------------------------

####三：使用无状态/无会话的设计

通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。通常我们在HTTP Session中保存着上下文数据，这种有状态的设计使得程序很难随着工作负载的增加而进行伸缩。比如某个服务实例拥有10000个会话的状态，则很难增加服务来分担其负荷，因为负荷被锁定在当前服务上了。如果程序被设计成一个无状态的，则可以自由增加服务实例，并且在这些实例之间平衡负载，从而使得服务具有较好的伸缩性，适用于大规模分布式系统。同时，由于服务器端不需要记录客户端的一系列Session，也减少了服务器端的性能开销。
无状态要求任意一个Web请求必须完全与其他请求隔离，所有的会话状态通过url的形式保存在了客户端，服务器端实现了无状态。

举个例子：一个心理测试的应用，需要用户做2次选择题，每次可选A、B两种答案，2次选择完毕之后将告知用户属于何种心理类型。
如果按ORB或SOA的服务思维，很容易想到在服务器端保存Session，每次选择以后修改Session，根据Session产生结果。但如果以REST的状态表述转移模型为指导，我们会自然地得出这样设计：

![无状态设计](http://images.cnblogs.com/cnblogs_com/weidagang2046/rest.PNG)

每一个页面表示一个状态（存在于客户端），页面包含了到下一个页面的超链接，每当用户选a或选b时分别转移到下一个相应的状态。这样，所有的会话状态其实都是通过url的形式保存在了客户端，服务器端实现了无状态。

-----------------------------

####四：使用XML或JSON来传输数据

数据中包含了对于资源的属性的描述，服务应该采取结构良好并且易于阅读的方式来描述资源。XML、JSON都是结构良好的语言，并且适于阅读。个人更推荐JSON。

####五：Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。Hypermedia API的设计被称为HATEOAS。前面提到的Github的API就是这种设计。如果想获取当前用户的信息，应该去访问`api.github.com/user`。这个设计理念其实跟第三点是一致的。

------------------------------

###《REST in Practice》是本好书

书中提到 Richardson 的 REST 成熟度模型，通过这个模型你可以理解什么是REST。这个模型是这样子的：

* 第0级服务：只使用一个URI作为一个服务端口，也只使用一个HTTP方法传输数据。大多数 WS-* 服务都是这个级别的，XML-RPC和POX也是。这种做法相当于把HTTP这个应用层协议降级为传输层协议用。HTTP头和有效载荷是完全隔离的，HTTP头只用于保证传输，不涉及业务逻辑；有效载荷包含全部业务逻辑，因此API可以无视HTTP头中的任何信息。

* 第1级服务：使用多个URI，不同的URI代表不同的调用入口，但只使用同一个HTTP方法传输数据。

* 第2级服务：使用多个URI，不同的URI代表不同的资源，同时使用多个HTTP方法操作这些资源，例如使用POST/GET/PUT/DELET 分别进行CRUD操作。这时候HTTP头和有效载荷都包含业务逻辑，例如 HTTP 方法对应CRUD操作，HTTP状态码对应操作结果的状态。我们现在看到的大多数RESTful API做到的也就是这个级别。

* 第3级服务：使用超媒体（hypermedia）作为应用状态引擎。

把超链接引入到多媒体当中去，那就得到了超媒体，关键还是超链接。使用超媒体作为应用引擎状态，意思是应用引擎的状态变更由客户端访问不同的超媒体资源驱动。使用超媒体与前面第 1 级、第 2 级的显著区别是，客户端不再和 URI 紧耦合。在第 1 级或者第 2 级的应用里面，客户端都需要知道资源使用的 URI 模版（如 /orders/{id}），然后要操作什么样的资源就生成什么样的 URI。超媒体客户端只知道入口 URI，之后的每一个 URI 都是通过超链接获得的。

-----------------------------

####参考文章

* [怎么样才算是 RESTful？](http://www.cnblogs.com/cathsfz/archive/2012/05/09/2493385.html)

* [深入浅出REST](http://www.infoq.com/cn/articles/rest-introduction#)

* [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
