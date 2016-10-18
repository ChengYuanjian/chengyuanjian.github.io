---
layout:         post
title:         Jekyll+Markdown中代码高亮
description: Jekyll原生支持语法高亮工具Pygments，Pygments支持多种语言高亮。这里介绍两种代码高亮的方法。
keywords: Markdown, Jekyll
category: Jekyll
tags: [Jekyll, Markdown]
---

{% include JB/setup %}

Jekyll原生支持语法高亮工具Pygments，Pygments支持多种语言高亮。这里介绍两种代码高亮的方法。

###1.Pygments

####1.1.优点

* 支持超过100种以上的代码高亮
* 不用担心代码格式化的问题，直接拷贝即可

<!-- more -->

####1.2.缺点

* 安装方式要难于highlight.js，前提要求已安装python
* 有新的版本，需要手动重新生成pygments.css

####1.3.安装方式

* `pip install Pygments`
* [点击下载](https://pypi.python.org/pypi/Pygments)安装包
`easy_install Pygments-1.6-py2.7.egg`

####1.4.生成pygments.css

`pygmentize -S default -f html > pygments.css`

####1.5.引入pygments.css

{% highlight html %}
<link rel="stylesheet" href="/pygments.css">
{% endhighlight %}

设置_config.yml中pygments=true

`pygments: true`

####1.6.用法

<pre><code>
{ % highlight language % }
   your code goes here  
{ % endhighlight % }
</code></pre>

或者
<pre><code>
```language
   your code goes here  
```
</code></pre>

用实际使用的lexers替换掉language即可，所有的lexers列表可参见[这里](http://pygments.org/docs/lexers/)

####1.7.官方资源

* [Pygments on github](https://github.com/mojombo/jekyll/wiki/Liquid-Extensions)
* [Pygments homepage](http://pygments.org/)
* [Supported languages list](http://pygments.org/languages/)

###2.Highlight.js

####2.1.优点

* 容易使用
* 方便升级

####2.2.缺点

* 代码高亮风格不如pygments友好，目前新版本已经漂亮很多了
* 需要自己组织代码格式

####2.3.引入highlight.js

1.引入在线资源

{% highlight html %}
<link rel="stylesheet" href="http://yandex.st/highlightjs/8.0/styles/default.min.css">
<script src="http://yandex.st/highlightjs/8.0/highlight.min.js"></script>
{% endhighlight %}

2.下载至本地后引入

下载地址：[点击](http://highlightjs.org/download/)

{% highlight html %}
<link rel="stylesheet" href="styles/default.css">
<script src="highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
{% endhighlight %}

####2.4.用法

代码用使用标签`<pre><code></code></pre>`包含即可。

####2.5.官方资源

* [Highlight.js homepage](http://softwaremaniacs.org/soft/highlight/en/)



