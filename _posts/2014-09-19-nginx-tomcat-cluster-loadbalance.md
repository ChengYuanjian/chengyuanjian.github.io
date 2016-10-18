---
layout:         post
title:         Nginx和Tomcat实现集群和负载均衡
description: 
keywords: Nginx, Tomcat
category: Java
tags: [Java, Nginx, Tomcat]
---

Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。说到Nginx和Apache的比较，在高并发连接的情况下，Nginx是Apache服务器最佳替代品。网上有一篇经典的博文，写得非常详细->[点击查看](http://zyan.cc/nginx_php_v6/)


_本文资料借鉴于网络_



####反向代理

* 反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。  

<!-- more -->

上次整理了[Apache和Tomcat实现集群和负载均衡](http://chengyuanjian.github.io/java/2014-09/apache-tomcat-cluster-loadbalance.html)的配置，本文简述一下Nginx和Tomcat实现集群和负载均衡。

####负载均衡

* 负载均衡（Load Balance），其意思就是将负载（工作任务）进行平衡、分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

F5是操作于OSI网络模型的传输层，Nginx、Apache是基于Http反向代理方式，位于OSI模型的第七层应用层，即TCP/UDP 和Http协议的区别。

----------------------------

####1.准备工作

* 下载[Nginx](http://nginx.org/en/download.html)

* 如果想实现SESSION共享，则需要利用memcached来保存session，以下是依赖包：

<pre><code>
http://memcached-session-manager.googlecode.com/files/memcached-session-manager-1.3.0.jar
 
http://memcached-session-manager.googlecode.com/files/msm-javolution-serializer-jodatime-1.3.0.jar
 
http://memcached-session-manager.googlecode.com/files/msm-javolution-serializer-cglib-1.3.0.jar
 
http://spymemcached.googlecode.com/files/memcached-2.4.2.jar

http://memcached-session-manager.googlecode.com/files/javolution-5.4.3.1.jar
</code></pre>

__如果tomcat过多不建议session同步，server间相互同步session很耗资源，高并发环境容易引起Session风暴。而且大多数情况下，同一个用户没有必要访问不同的server。__

####2.安装

同Apache一样简单，Windows下载后直接解压即可。如果需要memcached，将上述5个依赖包放到$TOMCAT_HOME/lib目录下。

Linux下载后执行以下命令：
{% highlight sh %}
tar -xzvf nginx-1.x.x.tar.gz 
cd nginx-1.x.x
./configure 
make 
make install　
{% endhighlight %}
如果是Ubuntu，常用在线安装的方式：`apt-get install nginx`。

####3.配置

* （1）修改<nginx_home>/nginx.conf文件(Linux下修改sites-available/default文件，方式基本雷同)：

{% highlight py %}
#Nginx所用用户和组，window下不指定
#user  niumd niumd;

#工作的子进程数量（通常等于CPU数量或者2倍于CPU）
worker_processes  2;

#错误日志存放路径
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
error_log  logs/error.log  info;

#指定pid存放文件
pid        logs/nginx.pid;

events {
    #使用网络IO模型linux建议epoll，FreeBSD建议采用kqueue，window下不指定。
    #use epoll;
    
    #允许最大连接数
    worker_connections  2048;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

		#定义日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] $request '
    #                  '"$status" $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  off;
    access_log  logs/access.log;

    client_header_timeout  3m;
    client_body_timeout    3m;
    send_timeout           3m;
 
    client_header_buffer_size    1k;
    large_client_header_buffers  4 4k;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    #keepalive_timeout  75 20;

    upstream tomcat {
      #根据ip计算将请求分配各那个后端tomcat，许多人误认为可以解决session问题，其实并不能。
      #同一机器在多网情况下，路由切换，ip可能不同
      #ip_hash; 通知nginx使用ip hash负载均衡算法。如果没加这条指令，nginx会使用默认的round robin负载均衡模块。
      server localhost:9080  weight=1; #weight为请求权重，值越大，被请求的机率越高。
      server localhost:8080  weight=1;
     }

    server {
      listen       80;
      server_name  localhost; 
      charset      utf-8;  

      location / { 
          root   html;  
          index  index.html index.htm; 
      	  proxy_connect_timeout   3;
      	  proxy_send_timeout      30;
      	  proxy_read_timeout      30;
          proxy_pass http://tomcat;     #代理上文配置的upstream tomcat
      }
      
      location ~ ^/(WEB-INF)/ {   
          deny all;   
      }   

      error_page   500 502 503 504  /50x.html;  
      location = /50x.html {  
        root   html;  
      } 
      
            
   }
}
{% endhighlight %}

主要改动在server里面，相当于一个代理服务器，可以配置多个。

*	listen：表示当前的代理服务器监听的端口，默认的是监听80端口。注意，如果我们配置了多个server，这个listen要配置不一样。

*	server_name：表示监听到之后需要转到哪里去，这时我们直接转到本地。

*	location：表示匹配的路径，这时配置了/表示所有请求都被匹配到这里。可以配置任意多个，采用最优匹配的原则进行代理分发；路径支持正则，一般以~开头$结尾（前面加*不区分大小写）。

*	root：里面配置了root这时表示当匹配这个请求的路径时，将会在这个文件夹内寻找相应的文件，这里对我们之后的静态文件伺服很有用。

*	index：当没有指定主页时，默认会选择这个指定的文件，它可以有多个，并按顺序来加载，如果第一个不存在，则找第二个，依此类推。

*	error_page是代表错误的页面

-----------------------

* （2）修改Tomcat的conf/service.xml文件，参考[这里](http://chengyuanjian.github.io/java/2014-09/apache-tomcat-cluster-loadbalance.html)。

解决SESSION共享，则还需修改Context节点：
{% highlight xml %}
<Context docBase="xx/WebContent" path="" reloadable="true" >
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="n1:localhost:11211 n2:localhost:11212"
    requestUriIgnorePattern=".*\.(png|gif|jpg|css|js)$"
    sessionBackupAsync="false"
    sessionBackupTimeout="100"
    transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"
    copyCollectionsForSerialization="false"
    />
</Context>
{% endhighlight %}

* （3）动静分离，即Nginx处理图片、html等静态的文件，Tomcat处理jsp动态文件。最主要用的还是location这个元素：

{% highlight sh %}
location ~ \.jsp$ {
        proxy_pass http://localhost:8080;
}
		
location ~ \.(html|js|css|png|gif)$ {
	root /opt/tomcat/webapps/ROOT;   
	expires 30d; 
}
{% endhighlight %}

* （4）运行`nginx -t`检查配置是否有误，`nginx -s reload`重新加载配置，`nginx -s stop`关闭服务。
