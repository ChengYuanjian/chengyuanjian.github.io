---
layout:         post
title:         Tomcat Classloader浅析
description: 本篇介绍Tomcat的classloader加载机制，基于Tomcat6.0版本。
keywords: Tomcat, Java
category: Java
tags: [Java,Tomcat]
---


##什么是Classloader?

Classloader即类加载器，把Java class加载到JVM里运行，负责加载Java class的这部分就叫Classloader。

##JVM的Classloader

Bootstrap Classloader即启动类加载器，是JVM实现的一部分（C++编写），在JVM启动的时候加载java核心API包括其他Classloader，ExtClassLoader加载java的扩展API，即lib/ext中的类，AppClassLoader加载用户CLASSPATH设置目录中的Class。

JVM采用双亲委托模式:
  {% highlight java%}
  if (parent != null) 
    parent.loadClass(name, false); 
  {% endhighlight %}
    
<!-- more -->

##Class.loadClass和Class.forName

通过loadClass加载类在加载的时候并不对该类进行解释，因此也不会初始化该类。使用forName加载的时候就会将Class进行解释和初始化。（关于JVM更多知识请参见《深入理解JVM》）

##Tomcat的Classloader

* Bootstrap - 载入JVM自带的类和$JAVA_HOME/jre/lib/ext/*.jar
* System - 载入$CLASSPATH/*.class
* Common - 载入$CATALINA_HOME/common/...，它们对TOMCAT和所有的WEB APP都可见
* Catalina - 载入$CATALINA_HOME/server/...，它们仅对TOMCAT可见，对所有的WEB APP都不可见
* Shared - 载入$CATALINA_HOME/shared/...，它们仅对所有WEB APP可见，对TOMCAT不可见
* WebApp - 载入ContextBase/WEB-INF/...，它们仅对该WEB APP可见

##线程上下文类加载器ContextClassloader

* 使用线程上下文类加载器, 可以在执行线程中, 抛弃双亲委派加载链模式, 使用线程上下文里的类加载器加载类.
* 使用Thread.currentThread().setContextClassLoader(...);更改当前线程的contextClassLoader，来改变其载入类的行为。
* 使用线程上下文加载类，要保证多个需要通信的线程间的类加载器应该是同一个，防止因为不同的类加载器，导致类型转换异常(ClassCastException).

##Tomcat的ContextClassloader

* 对于WEB APP线程，它的ContextClassLoader是WebAppClassLoader
* 对于Tomcat Server线程，它的contextClassLoader是CatalinaClassLoader

注意这里：WebAppClassloader的工作原理跟一般的双亲委托模式不同，它先试图自己载入类（WEN-INF/中载入类），如果无法载入，再请求父类完成。

##WebAppClassloader

抛弃了双亲委托模式，这样可以让不同的WEB APP之间的类载入互不干扰（同一个tomcat下可以部署多个app，不同app之间是不可见的）。
载入顺序为：
`WEB-INF/classes/*.class `
`WEB-INF/lib/ *.jar`
