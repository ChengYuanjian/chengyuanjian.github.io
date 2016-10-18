---
layout:         post
title:         关于iBatis中自定义数据源
description: 
keywords: Java, iBatis, MyBatis
category: Java
tags: [Java,iBatis,MyBatis]
---

本人接手了一个年代久远的老项目，DAO层是原始的JDBC直接调用拼接的SQL。一方面维护成本较高，另一方面也有SQL注入的风险，故计划用iBatis进行改造。原代码有一套比较完整的数据库连接池的方案，且数据库配置文件已加密来提高安全性。而iBatis自身也支持数据源配置。那该如何让iBatis读取原有的数据源，让二者完美结合呢？

<!-- more -->
前段时间写了一篇[关于iBatis多数据源、通配符配置](http://chengyuanjian.github.io/java/2014-09/ibatis-multiply-datasource.html)的文章，正文有一段代码
{% highlight java %}
sqlMapClient.setUserConnection(connection);
{% endhighlight %}
可以自定义Connection传给iBatis使用，可以解决这个问题，但在实际使用的过程中，依旧比较麻烦，因为你还得时刻操心Connection的open/close。对此，我一直耿耿于怀，经过一番实践，终于找到另外一种解决方案。

####1.配置iBatis数据源

{% highlight xml %}
  <transactionManager type="JDBC">
		<dataSource type="SIMPLE">
			<property name="JDBC.Driver" value="${db.driver}" />
			<property name="JDBC.ConnectionURL" value="${db.url}" />
			<property name="JDBC.Username" value="${db.usr}" />
			<property name="JDBC.Password" value="${db.pwd}" />
		</dataSource>
	</transactionManager>
{% endhighlight %}

####2.构造数据Properties

{% highlight java %}
Properties properties = new Properties();
properties.setProperty("db.driver","oracle.jdbc.driver.OracleDriver");
properties.setProperty("db.url","jdbc:oracle:thin:@localhost:1521:sid");//这里改为读取原有的数据源信息即可
properties.setProperty("db.usr","user");
properties.setProperty("db.pwd","password");//这里可以解密
{% endhighlight %}

####3.构造SqlMapClient

{% highlight java %}
  Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
  SqlMapClientBuilder.buildSqlMapClient(reader, properties);    
  reader.close();
{% endhighlight %}

到此结束。这里取了个巧，利用Properties保存原有的数据源信息，再构造出SqlMapClient。这样，原有的数据连接池可以正常使用，iBatis也不用再单独维护一份配置文件了，完美解决。
