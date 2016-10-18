---
layout:   post
title:    关于“ORA-01000：maximum open cursors exceeded”的解决方案
description: 
keywords: Oracle,cursor,01000
category: Oracle
tags: [Oracle]
---

####问题分析：

* 1.游标open后，出错了又没有close。

* 2.可能是表结构的问题。

####解决方法：

* 1.查看数据库当前的游标数配置：

{% highlight sql %}
show parameter open_cursors;
{% endhighlight %}
 
<!-- more -->

* 2.查看游标使用情况：

{% highlight sql %}
 select o.sid, osuser, machine, count(*) num_curs
 from v$open_cursor o, v$session s
 where user_name = 'user' and o.sid=s.sid
 group by o.sid, osuser, machine
 order by  num_curs desc;
{% endhighlight %} 

__此处的user_name='user'中,user代表占用数据库资源的数据库用户名。__
 

* 3.查看游标执行的sql情况：

{% highlight sql %}
 select o.sid q.sql_text
 from v$open_cursor o, v$sql q
 where q.hash_value=o.hash_value and o.sid = 123;
{% endhighlight %} 

__此处的o.sid = 123是步骤2中的结果集__

* 4.根据游标占用情况分析访问数据库的程序在资源释放上是否正常,如果程序释放资源没有问题，则加大游标数。

{% highlight sql %}
alter system set open_cursors=2000 scope=both;
{% endhighlight %} 
