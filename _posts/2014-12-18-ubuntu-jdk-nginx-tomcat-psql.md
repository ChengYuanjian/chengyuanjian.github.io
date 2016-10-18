---
layout:         post
title:         Ubuntu基础环境搭建
description: 本篇介绍Ubuntu基础环境搭建
keywords: Unix, Linux, Ubuntu
category: Unix
tags: [Unix,Linux,Ubuntu]
---

本文基于Ubuntu12.04，阐述基础环境搭建过程：JDK+Nginx+Tomcat+Redis+PostgreSQL/MySQL。

####JDK安装

* 1.在线安装

Linux系统基本都可以使用`wget`命令在线安装
{% highlight sh %}
wget -c http://download.oracle.com/otn-pub/java/jdk/7/jdk-7-linux-x64.tar.gz  
{% endhighlight %}
但一直没有成功，也没有找到其他合适的链接，故只好作罢。尝试了另外一种方式，如下：

{% highlight sh %}
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
sudo apt-get install oracle-java7-set-default
{% endhighlight %}

<!-- more -->

* 2.离线安装

Oracle官方网站[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)安装包，
解压即可：`tar zxvf jdk-7u5-linux-x64.tar.gz -C /usr/lib/jvm`

接下来就是环境变量配置了。
{% highlight sh %}
gedit /etc/profile 

#追加以下配置：
export JAVA_HOME=/usr/lib/jvm/java-7-sun  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH 

#使设置生效
source /etc/profile 
{% endhighlight %}

* 3.Open JDK安装

{% highlight sh %}
apt-get install openjdk-6-jre
{% endhighlight %}
这种方式最为简单，使用过一段时间的OpenJDK，暂未发现与Oracle JDK有何不同，也未碰见兼容问题。
{% highlight sh %}
update-alternatives --config java  
{% endhighlight %}
可以列出各种JDK版本，使用以下命令设置默认JDK版本。
{% highlight sh %}
update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-sun/bin/java 300  
update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-7-sun/bin/javac 300  
{% endhighlight %}

####Nginx安装

* 1.在线安装
{% highlight sh %}
apt-get install nginx
{% endhighlight %}

所有的配置文件都在/etc/nginx下，并且每个虚拟主机已经安排在了/etc/nginx/sites-available下
启动程序文件在/usr/sbin/nginx
日志放在了/var/log/nginx中，分别是access.log和error.log
并已经在/etc/init.d/下创建了启动脚本nginx
默认的虚拟主机的目录设置在了/usr/share/nginx/www

* 2.离线安装

下载地址：http://nginx.org/download/.解压后，进入目录，执行以下命令：
{% highlight sh %}
./configure
make
make install
{% endhighlight %}
安装成功之后，nginx放置在/usr/local/nginx目录下，主要的配置文件为conf目录下的nginx.conf，nginx的启动文件在sbin目录下的nginx文件。

####Tomcat安装

因为需要做tomcat集群，故直接在官网下载压缩包：http://tomcat.apache.org/，解压即可。

####Redis安装

* 1.在线安装

{% highlight sh %}
wget http://download.redis.io/releases/redis-2.8.3.tar.gz  
$ tar xzf redis-2.8.3.tar.gz  
$ cd redis-2.8.3  
$ make  
$ sudo make install   #这时Redis的可执行文件被放到了/usr/local/bin  
{% endhighlight %}

或者
{% highlight sh %}
apt-get install redis-server
{% endhighlight %}

* 2.离线安装

与在线安装方式一基本一样，不赘述。

####PostgreSQL安装


* 1.在线安装
 
{% highlight sh %}
apt-get install -y postgresql-9.1 postgresql-client-9.1 postgresql-contrib-9.1 postgresql-server-dev-9.1 
{% endhighlight %}
或者直接
{% highlight sh %}
apt-get install postgresql
{% endhighlight %}

参见：http://www.postgresql.org/download/linux/ubuntu/

####MySQL安装

* 1.在线安装

{% highlight sh %}
apt-get install mysql-server
{% endhighlight %}
配置文件路径：/etc/mysql/my.cnf


