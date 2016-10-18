---
layout: post
title:  详解Forward和Redirect
description: 
keywords: Java,HTTP
category: Java
tags: [Java,HTTP]
---


用户向服务器发送了一次HTTP请求，该请求会经过多个信息资源处理以后才返回给用户，各个信息资源使用请求转发机制相互转发请求，用户是感觉不到请求转发的。根据转发方式的不同，可以区分为直接请求转发(Forward)和间接请求转发(Redirect)两种。
那么两者之间的区别是什么呢？

<!-- more -->

####1.从地址栏显示来说 

forward是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,所以它的地址栏还是原来的地址.

redirect是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

####2.从数据共享来说 

forward:转发页面和转发到的页面可以共享request里面的数据.

redirect:不能共享数据.

####3.从运用地方来说 

forward:一般用于用户登陆的时候,根据角色转发到相应的模块.

redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等.

####4.从效率来说 

forward:高.

redirect:低.


下面是详细解析：

--------------------------------

###直接请求转发(Forward)

Web应用程序大多会有一个控制器。由控制器来控制请求应该转发给那个信息资源。然后由这些信息资源处理请求，处理完以后还可能转发给另外的信息资源来返回给用户，这个过程就是经典的MVC模式。
`javax.serlvet.RequestDispatcher`接口是请求转发器必须实现的接口，由Web容器为Servlet提供实现该接口的对象，通过调用该接口的forward()方法到达请求转发的目的，示例代码如下：

{% highlight java %}
 public void doGet(HttpServletRequest request , HttpServletResponse response){
     //获取请求转发器对象，该转发器的指向通过getRequestDisPatcher()的参数设置
   RequestDispatcher requestDispatcher =request.getRequestDispatcher("URL");
    //调用forward()方法，转发请求      
   requestDispatcher.forward(request,response);    
}
{% endhighlight %}

![Forward](http://img2.tuicool.com/uUr6VrJ.png)

上图所示的直接转发请求的过程如下：

* 1.浏览器向Servlet1发出访问请求；

* 2.Servlet1调用forward()方法，在服务器端将请求转发给Servlet2；

* 3.最终由Servlet2做出响应。

客户端浏览器只发出一次请求，Servlet把请求转发给Servlet、HTML、JSP或其它信息资源，由第2个信息资源响应该请求，两个信息资源*共享同一个request对象*。

--------------------------------

###间接请求转发(Redirect)

间接转发方式，有时也叫重定向，它一般用于避免用户的非正常访问。例如：用户在没有登录的情况下访问后台资源，Servlet可以将该HTTP请求重定向到登录页面，让用户登录以后再访问。在Servlet中，通过调用response对象的SendRedirect()方法，告诉浏览器重定向访问指定的URL，示例代码如下：

{% highlight java %}
public void doGet(HttpServletRequest request,HttpServletResponse response){
//请求重定向到另外的资源
    response.sendRedirect("资源的URL");
}
{% endhighlight %}

![Redirect](http://img0.tuicool.com/aa6VZf3.png)

上图所示的间接转发请求的过程如下：

* 1.浏览器向Servlet1发出访问请求；

* 2.Servlet1调用sendRedirect()方法，将浏览器重定向到Servlet2；

* 3.浏览器向servlet2发出请求，最终由Servlet2做出响应。 

服务器端在响应第一次请求的时候，让浏览器再向另外一个URL发出请求，从而达到转发的目的。它本质上是两次HTTP请求，对应两个request对象。
