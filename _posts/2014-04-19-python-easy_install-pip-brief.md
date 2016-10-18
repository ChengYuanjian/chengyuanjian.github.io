---
layout: post
title: easy_install和pip简介
description: 
keywords: Python, pip, easy_install
category: Python
tags: [pip,easy_install]
---

`easy_install`的作用和perl中的`cpan`, ruby中的`gem`类似，都提供了在线一键安装模块的傻瓜方便方式，而`pip`是`easy_install`的改进版, 提供更好的提示信息，删除package等功能。老版本的python中只有`easy_install`, 没有`pip`。

###easy_install的用法：

* 安装一个包
{% highlight sh%}
 $ easy_install <package_name>
 $ easy_install "<package_name>==<version>"
{% endhighlight %}

* 升级一个包
{% highlight sh%}
 $ easy_install -U "<package_name>>=<version>"
{% endhighlight %}

<!-- more -->

###pip的用法

* 安装一个包
{% highlight sh%}
 $ pip install <package_name>
 $ pip install <package_name>==<version>
{% endhighlight %}

* 升级一个包 (如果不提供version号，升级到最新版本）
{% highlight sh%}
 $ pip install --upgrade <package_name>>=<version>
{% endhighlight %}

* 删除一个包
{% highlight sh%}
 $ pip uninstall <package_name> 
{% endhighlight %}
