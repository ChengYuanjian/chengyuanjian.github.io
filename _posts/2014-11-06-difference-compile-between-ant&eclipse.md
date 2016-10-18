---
layout:         post
title:         关于编译Java在Eclipse下通过，ANT报错的解决方案
description: 本文阐述了ANT编译和Eclipse编译的不同点
keywords: ANT,Eclipse,Java
category: ANT
tags: [ANT,Eclipse]
---

Eclipse是写Java首选IDE，而ANT做为Java的一种build工具，二者可以完美的结合。但偶尔我们遇到这样一个奇怪的现象：同样的代码，在Eclipse下编译一切正常，但用ANT脚本编译打包的时候会报错。这是什么原因呢？本文将解释问题的根源，并提供解决方案。

<!-- more -->

####二者编译方式不同

* ANT编译是通过`javac`节点，调用JDK下的`javac`来实现的。

* Eclipse编译是通过自身集成的JRE实现，但并非直接去调用，而是通过JDTCompilerAdapter进行适配的。

####解决方案

既然已经知道二者编译方式的不同，那就知道解决方案了。能否让ANT也通过JDTCompilerAdapter进行编译？答案是肯定的。

* 1.增加`build.compiler`属性

{% highlight xml %}
<property name="build.compiler" value="org.eclipse.jdt.core.JDTComilerAdapter"/>
{% endhighlight %}

* 2.导入相关jar包

把eclipse/plugins目录下的`org.eclipse.jdt.*.jar`复制到ANT安装目录下的lib文件夹中即可。

__如果依旧报"org.eclipse.jdt.core.JDTComilerAdapter not found"，则把org.eclipse.jdt.core_*.jar中的jdtCompilerAdapter.jar提取出来放到lib目录下__

* 3.修改`javac`节点

{% highlight xml %}
<javac srcdir="${src.dir}" destdir="${classes.dir}" debug="true" deprecation="true" source="1.6" target="1.6">
    <classpath refid="jar.classpath">
    </classpath>
</javac>
{% endhighlight %}
这里的`source`和`target`属性必须指定，否则也会报错。

* 4.配置ANT的runtime

如果是使用命令行的方式运行ANT，则上述3步即可。如果需要在Eclipse中运行ANT，则还需要更改其runtime配置。
`Window-Preferences-Ant-Runtime-Ant Home Entries-Add External Jars`，把步骤2中的jar包都导入进来即可。

---------------------

####其他文章

* [ANT脚本示例教程](http://chengyuanjian.github.io/ant/2014-08/ant-build.html)

* [利用ANT生成JUnit、Findbugs、Emma报告](http://chengyuanjian.github.io/ant/2014-08/ant-junit-findbugs-emma.html)
