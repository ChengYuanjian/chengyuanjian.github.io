---
layout: post
title: 程序员那点儿破事儿
description: 
keywords: programmer
category: Other
tags: [programmer,Python]
---

###去死吧！你这个异教徒！ 

<pre><code>一天我路过一座桥，碰巧看见一个人想跳河自杀。我跑过去对他大喊道：“别跳，别死啊。”
“为什么不让我跳？”他说。
“因为还有很多东西值得我们活下去啊。”
“有吗？比如说？”
“呃……你做什么工作？”
“程序员。”
我说：“我也是！瞧，有共同点了吧。你是软件还是硬件？”
“软件。”
“我也是！PC 还是 Web？”
“PC。”
“我也是！Windows 还是 Linux 平台？”
“Linux。”
“我也是！那你使用 Java 还是 C++？”
“C++。”
“我也是。1998 年的 C++ 98 还是 2011 年的 C++ 11？”
“2011 年的 C++ 11。”
“我也是。大括号写在后面还是写在下一行？”
“下一行。”
“去死吧！你这个异教徒人渣！”我一把将他推下桥去。
</code></pre>

<!-- more -->

以上是网上一个段子，有人跟帖—————

“要是python还可以救人一命。”

“你们这些用python的，应该被钉在十字架上烧死。”

###程序员和禅师

<pre><code>
程序员问禅师：我总是和我的同事们合不来，他们使用 Java，我使用 PHP。

禅师笑而不语，拿出一根筷子，递给青年：来，折断它。

青年接过筷子，很轻松地就折断了。

禅师又拿出四十七根筷子，青年抢过来，抄出把斧头全砍断了。

禅师沉吟片刻，摆出架式，双手合十：你们应该使用 Python 语言。
</code></pre>

###为何？

作为一个接触java七年有余的IT Farmer来说，看到Python第一印象会是...

* “卧槽代码还可以这么写！”

* “我擦劳资真的不用写大括号他也知道我说的是啥吗？”

* “尼玛以前几十行代码，现在几行搞定？”

等等，诸如此类。

随意看看几个小例子，对比对比吧。

####交换变量

* Java

{% highlight java %}
temp = a
a = b
b = temp
{% endhighlight %}

* Python

{% highlight py %}
b, a = a, b
{% endhighlight %}

####倒序

* Java

{% highlight java %}
public static String reverseSort(String str) {
		String str2 = "";
		for (int i = str.length() - 1; i > -1; i--) {
			str2 += String.valueOf(str.charAt(i));
		}

		return str2;
	}
{% endhighlight %}

* Python

{% highlight py %}
str[::-1]
{% endhighlight %}


####文件读写

* Java

{% highlight java %}
//读
try {
	reader = new BufferedReader(new FileReader(file));
	String line;
	while ((line = reader.readLine()) != null) {
	content += line;
	}
	if (reader != null) {
	try {
		reader.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	} catch (Exception e) {
	e.printStackTrace();
}

//写
try {
	FileWriter writer = new FileWriter(fileName, true);
	writer.write(content);
	writer.close();
	} catch (IOException e) {
            e.printStackTrace();
        }
{% endhighlight %}

* Python

{% highlight py %}
#读
f=open(fileName,'r')
f.close()

#写
f=open(fileName,"w")
f.write(line)
f.close()
{% endhighlight %}

看到差异了吗？类似这样的很多，数不胜数。

当然，并非Python全面领先Java，Java依旧是企业应用中的王者，快速开发中小型应用，Python真心好用很多。光变量声明不需要指定类型和不需要加`;`，已经让我觉得很赞了。
我的建议是，除了Java\C++之外，至少掌握一门Python、Ruby，自己选吧。


