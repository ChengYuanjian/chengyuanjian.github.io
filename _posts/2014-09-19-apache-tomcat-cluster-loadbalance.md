---
layout:         post
title:         Apache和Tomcat实现集群和负载均衡
description: 
keywords: Apache, Tomcat
category: Java
tags: [Java, Apache, Tomcat]
---

Apache是世界使用排名第一的Web服务器软件，但只支持静态网页。Tomcat是由Apache软件基金会下属的Jakarta项目开发的一个Servlet容器。由于Tomcat本身也内含了一个HTTP服务器，它也可以被视作一个单独的Web服务器。换句话说，Apache是web服务器，Tomcat是应用（java）服务器，一个servlet容器。Apache，Nginx，Tomcat并称为Web服务三剑客。本文讲述如用利用Apache和Tomcat来实现集群和负载均衡。

_本文资料借鉴于网络_

<!-- more -->

-----------------

####1.准备工作

首先自然是要下载相关软件：

* Tomcat：[apache-tomcat-6.0.35](http://labs.mop.com/apache-mirror/tomcat/tomcat-6/v6.0.35/bin/apache-tomcat-6.0.35-windows-x86.zip)

* Apache：[httpd-2.2.22-win32-x86-no_ssl](http://labs.mop.com/apache-http://apache.etoak.com//httpd/binaries/win32/httpd-2.2.22-win32-x86-no_ssl.msi)

* jk for windows：[tomcat-connectors-1.2.32-windows-i386-httpd-2.2.x.zip](http://archive.apache.org/dist/tomcat/tomcat-connectors/jk/binaries/windows/) 必要插件，Apache必须通过该插件方可访问Tomcat

* [jk for linux](http://archive.apache.org/dist/tomcat/tomcat-connectors/jk/binaries/linux/jk-1.2.31/i386/)

####2.安装

按照方式比较简单，直接解压即可，然后把JK插件复制到apache/modules目录下。

####3.配置

* （1）修改<apache_home>/conf/httpd.conf文件，在文件后面加上：
{% highlight sh %}
LoadModule jk_module modules/mod_jk.so #JK插件的位置 
JkWorkersFile conf/workers.properties #tomcat相关配置文件，见下文
JkLogFile logs/mod_jk.log #日志文件
JkLogLevel debug  #tomcat运行模式
JkMount /*.jsp loadbalancer   #收到.jsp结尾的文件交给负载均衡器处理
JkMount /myapp/* loadbalancer  #收到myapp/路径交给负载均衡器处理
{% endhighlight %}

* （2）在<apache_home>/conf目录下创建：workers.properties文件：
{% highlight properties %}
worker.list= worker1,worker2,loadbalancer        #apache把Tomcat看成是worker，loadbalancer是负载均衡器

worker.worker1.host=localhost        #Tomcat worker1服务器
worker.worker1.port=8009            #Tomcat端口
worker.worker1.type=ajp13            #协议
worker.worker1.lbfactor=100            #负载平衡因数
 
worker.worker2.host=localhost        #Tomcat worker2服务器<pre><code>
worker.worker2.port=8010            #如果是一台机器上端口不能一样，常识
worker.worker2.type=ajp13            #协议
worker.worker2.lbfactor=100            #设为一样代表两台机器的负载相同
 
worker.loadbalancer.type=1b
worker.loadbalancer.balanced_workers=worker1,worker2
worker.loadbalancer.sticky_seesion=false
worker.loadbalancer.sticky_session_force=false
{% endhighlight %}

__说明：__

1.JK插件的负载均衡器根据配置的lbfactor（负载平衡因数），负责为集群系统中的Tomcat服务器分配工作负荷，以实现负载平衡，值越大，负载就越高。每个Tomcat服务器间用集群管理器（SimpleTcpCluster）进行通信，以实现HTTP回话的复制，比如Session。
2.worker.loadbalancer.sticky\_seesion如果设为true则说明会话具有“粘性”，也就是如果一个用户在一个Tomcat中建立了会话后则此后这个用户的所有操做都由这个Tomcat服务器承担。集群系统不会进行会话复制。如果设为false则下面的 sticky\_session\_force无意义。
3.sticky\_session\_force：假设sticky\_session设为true，用户会话具有了粘性，当当前Tomcat服务器停止服务后，如果sticky\_session\_force为true也就是强制会话与当前Tomcat关联，那么会报500错误，如果设为false则会转到另外的Tomcat服务器。

* （3）修改两个Tomcat的conf/service.xml文件：

确保SHUTDOWN端口、HTTP端口、AJP端口不冲突（这里AJP端口与上述workers.properties中配置保持一致）；

在Engine节点启用集群配置，去掉Cluster节点前的注释（默认是注释掉的），并添加jvmRoute属性（与上述workers.properties中配置保持一致）；
{% highlight xml %}
<Engine name="Catalina" defaultHost="localhost" jvmRoute="worker1">
  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/> 
{% endhighlight %}

要实现session复制，还需要在context.xml添加属性distributable="true"：
{% highlight xml %}
<Context distributable="true">
{% endhighlight %}
或者在应用程序的web.xml中添加<distributeable/>节点。Tomcat服务器启动这个Web应用时，会为它创建由<Cluster>元素指定的会话管理器。

####4.写在最后

通过这种方式配置，如果一个用户A发送请求到worker1，如果中途worker1 down掉了，会转向worker2。如果中途worker3启动了，也有可能请求到worker3，但不管worker1是否重新启动，用户A的请求不会再次到worker1，除非用户重新登陆。
