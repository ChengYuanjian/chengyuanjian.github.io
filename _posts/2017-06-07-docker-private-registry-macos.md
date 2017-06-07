---
layout: post
title:  Docker私有Registry搭建（Mac OS）
description: 
keywords: Docker
category: Docker
tags: [Docker,Mac OS]
---

Docker的私有仓库类似maven的私服，一般用于公司或个人搭建一个类似docker hub的环境，这样上传、下载镜像速度较快，自由度也较高。本文以Mac为例，阐述如何搭建Docker私有仓库。

<!-- more -->

### 1.安装docker

Docker官方已经支持Mac和Windows，下载地址：https://www.docker.com/docker-mac
具体安装过程：https://github.com/widuu/chinese_docker/blob/master/installation/mac.md （非常详细，这里不赘述了）

### 2.开始搭建

在完成步骤1之后，默认会创建一个名字为default的虚拟机来运行docker，分配的IP默认为192.168.99.100，你可以使用以下命令来获取：
{% highlight sh %}
docker-machine ip default
{% endhighlight %}

#### 2.1修改docker启动配置

由于docker最新版本默认访问私服时，强制采用SSL安全连接。即与docker registry交互默认使用的是https，然而我们搭建的私有仓库只提供http服务，如果不做特殊的配置，就会报类似于`dial tcp 192.168.99.100:5000: connection refused. `错误，配置如下：
{% highlight sh %}
docker-machine ssh default #登陆虚拟机
sudo vi /var/lib/boot2docker/profile #修改配置
{% endhighlight %}

加入`EXTRA_ARGS="--insecure-registry 192.168.99.100:5000"`，然后重启虚拟机：
{% highlight sh %}
docker-machine restart default
{% endhighlight %}

验证是否生效：
{% highlight sh %}
docker info
{% endhighlight %}

最后的insecure registries节点如果显示配置的信息，则表示成功。

#### 2.2创建私服容器

##### 先下载registry镜像

{% highlight sh %}
docker pull registry 
{% endhighlight %}
注意：可能是网络的问题，经常会失败，可以多尝试几次（本人8次后方才成功）

##### 通过该镜像启动容器

默认情况下，会将仓库存放于容器内的/tmp/registry目录下。如果容器被删除，则存放于容器中的镜像也会丢失，所以我们一般情况下会指定本地一个目录挂载到容器内的/tmp/registry：
{% highlight sh %}
docker run -d -p 5000:5000 -v /Users/chengyuanjian/Documents/docker/registry:/tmp/registry registry
{% endhighlight %}
到此搭建完毕。

### 3.测试

私有Registry搭建完成后，我们需要测试一下是否可用。

* 先随意拉取一个镜像，这里使用经典的hello-world，主要原因是很小（1k左右）

{% highlight sh %}
docker pull hello-world
{% endhighlight %}
这里可以使用本地任意镜像。

* 打标签成私服镜像

{% highlight sh %}
docker tag hello-world 192.168.99.100:5000/hello-world
{% endhighlight %}

* 上传到私有仓库

{% highlight sh %}
docker push 192.168.99.100:5000/hello-world
{% endhighlight %}

* 删除本地镜像

{% highlight sh %}
docker rmi -f hello-world 192.168.99.100:5000/hello-world
{% endhighlight %}

* 从私库拉取镜像

{% highlight sh %}
docker pull 192.168.99.100:5000/hello-world
{% endhighlight %}

* 验证拉取是否成功

{% highlight sh %}
docker images
{% endhighlight %}
