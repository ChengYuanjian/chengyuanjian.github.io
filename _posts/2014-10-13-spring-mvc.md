---
layout: post
title:  Spring MVC
description: 
keywords: Spring,MVC
category: Spring
tags: [Spring,MVC]
---

继续上次关于REST的话题，[REST简述](http://chengyuanjian.github.io/other/2014-10/rest-brief.html)对REST设计思想做了一个简单的介绍。那么，实际应用中，如何构造Restful的Web服务呢？在 Java™ 中，可以使用以下几种方法来创建 RESTful Web Service：

* 使用 JSR 311（311）及其参考实现Jersey

* 使用 Restlet 框架

* Spring 3.0即以后版本，REST 支持被无缝整合到 Spring 的 MVC 中了

本文先介绍使用 Spring MVC。

<!-- more -->

---------------------

####Spring MVC是什么

首先要理解Spring MVC，它是基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，类似Struts。

* 前端控制器是DispatcherServlet

* 应用控制器分为处理器映射器(Handler Mapping)将请求映射到处理器，处理器适配器（HandlerAdapter）支持任意的类作为处理器，视图解析器(View Resolver)进行视图管理

* 页面控制器/处理器为Controller接口的实现，也可以是任何的POJO类

####核心流程

* 1.客户端发送请求，DispatcherServlet作为统一访问点，进行全局的流程控制

* 2.DispatcherServlet委托HandlerMapping， HandlerMapping把请求映射为HandlerExecutionChain对象（包含一个Handler处理器、多个HandlerInterceptor拦截器）对象

* 3.DispatcherServlet委托HandlerAdapter，HandlerAdapter把处理器包装为适配器，从而支持多种类型的处理器

* 4.HandlerAdapter根据适配的结果调用真正的处理器的方法，完成功能处理并返回一个ModelAndView对象（包含模型数据、逻辑视图名）

* 5.ViewResolver把逻辑视图名解析为具体的View

* 6.View根据传进来的Model模型数据进行渲染

* 7.返回控制权给DispatcherServlet，由DispatcherServlet返回响应给客户端

---------------------------

####一.前端控制器DispatcherServlet配置

其本质就是一个Servlet，在web.xml中配置：

{% highlight xml %}
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 自定义配置文件，多个以“,”分割，不指定会加载默认配置 -->
        <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
{% endhighlight %}

DispatcherServlet会默认加载WEB-INF/[DispatcherServlet的Servlet名字]-servlet.xml配置文件，这里会加载springmvc-servlet.xml。

#####url-pattern配置小结

<pre><code>
配置servlet的<url-pattern>时，容器会首先查找完全匹配，再查找目录匹配，最后查找扩展名匹配。 如果一个请求匹配多个“目录匹配”，容器会选择最长的匹配。

（1）“*.action”这是比较传统的方式，简单实用；
（2）“/”是rest风格的方式，后面会具体提到；
（3）“/*”请求虽然会顺利发送到Controller，但跳转到jsp页面时会被拦截，这种方式不可取。
</code></pre>

使用Spring构造restful url，通常会配置为`/`，但这种方式也会把js、jpg、css等静态资源拦截住，导致页面无法加载这些资源。这里有三种解决方案(推荐第二种)：

* 1.使用mvc:default-servlet-handler 

它会把url注册到SimpleUrlHandlerMapping的urlMap中,把对静态资源的访问由HandlerMapping转到`org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler`处理。DefaultServletHttpRequestHandler使用就是各个Servlet容器自己的默认Servlet。

* 2.使用mvc:resources 

{% highlight xml %}
<mvc:resources mapping="/images/**" location="/images/" />  
{% endhighlight %}

/images/**映射到ResourceHttpRequestHandler进行处理，location指定静态资源的位置。

* 3.在DispatcherServlet之前，让容器default的Servlet先拦截

{% highlight xml %}
<servlet-mapping>   
    <servlet-name>default</servlet-name>  
    <url-pattern>*.jpg</url-pattern>     
</servlet-mapping>    
<servlet-mapping>       
    <servlet-name>default</servlet-name>    
    <url-pattern>*.js</url-pattern>    
</servlet-mapping>  
{% endhighlight %}

#####常见容器默认Servlet名字：

<pre><code>
Tomcat, Jetty, JBoss, GlassFish：default
Google App Engine：_ah_default
Resin为：resin-file
WebLogic：FileServlet
WebSphere：SimpleFileServlet
</code></pre>

如果配置listener来加载配置文件，Spring会创建一个全局的WebApplicationContext上下文，称为根上下文（root application context）：
{% highlight xml %}
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
   
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:config/applicationContext.xml</param-value>
</context-param>
{% endhighlight %}

__与DispatcherServlet不同的是,DispatcherServlet可以同时配置多个，每个DispatcherServlet有一个自己的WebApplicationContext上下文，相互之间不可见。这个上下文继承了根上下文中所有东西。__

----------------------

####二.配置HandlerMapping、HandlerAdapter、HandlerInterceptor 

{% highlight xml %}
<!-- HandlerMapping，可以配置多个interceptor -->
<bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping">
 <property name="interceptors">       
     <list>       
         <bean class="com.mvc.MyInteceptor"></bean>      
     </list>       
 </property>    
</bean>

<!-- HandlerAdapter -->
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/> 
{% endhighlight %}

`<mvc:annotation-driven />`是一种简写形式，自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter，即与上述配置等价，但自此就无法指定Interceptor了。

#####HandlerMapping接口常用实现类： 

* BeanNameUrlHandlerMapping：表示将请求的URL和Bean名字映射，如URL为 “context/cyj”，则Spring配置文件必须有一个名字为“/cyj”的Bean。

* SimpleUrlHandlerMapping：通过配置文件，把一个URL映射到Controller。

* DefaultAnnotationHandlerMapping(3.0)/RequestMappingHandlerMapping(3.1)：通过注解，把一个URL映射到Controller类上。

__多个HandlerMapping的执行顺序问题，order值比较小的会优先执行__

* DefaultAnnotationHandlerMapping的order属性值是：0 

* `<mvc:resources/>`自动注册的 SimpleUrlHandlerMapping的order属性值是：2147483646 

* `<mvc:default-servlet-handler/>`自动注册的SimpleUrlHandlerMapping的order属性值是：2147483647 


#####HandlerAdapter接口常用实现类：

* SimpleControllerHandlerAdapter：表示所有实现了org.springframework.web.servlet.mvc.Controller接口的Bean可以作为Spring Web MVC中的处理器。

* AnnotationMethodHandlerAdapter(3.0)/RequestMappingHandlerAdapter(3.1)：通过注解，把一个URL映射到Controller类的方法上。

#####HandlerInterceptor：一般我们会使用抽象类`org.springframework.web.servlet.handler.HandlerInterceptorAdapter`

HandlerInterceptor是一个接口，一般需要自己扩展该接口，除了指定给HandlerMapping外，还可以全局配置：
{% highlight xml %}
<!-- 全局配置1  --> 
<mvc:interceptors>  
    <bean class="com.mvc.MyInteceptor" />  
</mvc:interceptors>  

<!-- 全局配置2，拦截匹配的URL  --> 
<mvc:interceptors >    
  <mvc:interceptor>    
      <mvc:mapping path="/cyj/*" />    
      <bean class="com.mvc.MyInteceptor"></bean>    
    </mvc:interceptor>    
</mvc:interceptors>  
{% endhighlight %}

同样，`<mvc:interceptors/>`是一种简写形式，与上述配置1等价。

-----------------------

####三.配置ViewResolver

{% highlight xml %}
<!-- ViewResolver -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
{% endhighlight %}

viewClass：JstlView表示JSP模板页面需要使用JSTL标签库
prefix和suffix：查找视图页面的前缀和后缀，如果逻辑视图名为cyj，则该jsp视图页面应该存放在“WEB-INF/jsp/cyj.jsp”

#####常用实现类

* UrlBasedViewResolver：通过配置文件，把一个视图名交给到一个View来处理 

* InternalResourceViewResolver：加入了JSTL的支持 

---------------------

####四.处理器/页面控制器Controller

在2.5之前，通过实现`org.springframework.web.servlet.mvc.Controller`接口定义处理器类。2.5引入注解式处理器支持，通过`@Controller`和 `@RequestMapping`注解定义处理器类（需要注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter）。
{% highlight java %}
/*响应http://host:port/context/user/cyj1或者cyj2请求*/
@Controller 
@RequestMapping(value="/user")         
public class MyController {
    @RequestMapping(value = {"/cyj1","cyj2"}) //可配置多个映射                 
    public ModelAndView cyj() {
    ModelAndView mv = new ModelAndView();
    //添加模型数据 可以是任意的POJO对象
    mv.addObject("message", "I'm cyj");
    //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
    mv.setViewName("cyj");
    return mv;                                         
  }
}
{% endhighlight %}

--------------------

####五.视图页面

一般为使用JSTL标签库的jsp页面，不赘述。

####六.其他

#####1.中文乱码解决方案

{% highlight xml %}
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

#####2.自动检测组件

{% highlight xml %}
<context:component-scan base-package="com.cyj.controller"/>
{% endhighlight %}

#####3.国际化

{% highlight xml %}
<bean id="messageSource"
  class="org.springframework.context.support.ResourceBundleMessageSource"
  p:basename="message">
</bean>
{% endhighlight %}

Spring会自动搜索message.properties、message\_zh\_CN.properties等国际化配置文件。
在页面中，引入Spring标签库：
{% highlight jsp %}
<%@taglib uri="http://www.springframework.org/tags" prefix="spring" %>
<spring:message code="key" />
{% endhighlight %}

如果有多个配置文件：
{% highlight xml %}
  <property name="basenames">
    <list>
      <value>message01</value>
      <value>message02</value>
      <value>message03</value>
    </list>
  </property>
{% endhighlight %}

------------------------

####参考文档

[Spring官方在线文档库](http://docs.spring.io/spring/docs/)
[Spring MVC 教程](http://elf8848.iteye.com/blog/875830/)
