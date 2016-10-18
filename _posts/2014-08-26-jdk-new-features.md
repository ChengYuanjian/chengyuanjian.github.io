---
layout:         post
title:         JDK各版本很cool的特性
description: 
keywords: Java, JDK
category: Java
tags: [Java,JDK]
---

私以为自从Sun被Oracle收购之后，Java便开始不思进取，虽然Java在企业应用领域的霸主地位无法撼动，但在Web领域内，前有PHP，后有Python、Ruby、nodeJS。而期待已久的Java 8的发布整整推迟了2年的时间，让人不得不怀疑Oracle对Java的用心程度。

Java9正在紧锣密鼓的开发中，新一代的Java采用了模块化的设计。虽然还未给出发布日期，但官方提前发布了Java SE9的特性。希望这次不要让我们等太久。

<!-- more -->

虽然JDK每个新版本的发布不如C#来得更夺人眼球，但确实有一些很cool的新特性，但这远远不够：

###JDK5

JDK5（代号“猛虎”）的一个重要主题就是通过新增一些特性来简化开发，这些特性包括泛型，for-each循环，自动装包/拆包，枚举，可变参数, 静态导入。使用这些特性有助于我们编写更加清晰，精悍，安全的代码。

####1.泛型(Generic)

C++通过模板技术可以指定集合的元素类型，而Java在1.5之前一直没有相对应的功能。一个集合可以放任何类型的对象，相应地从集合里面拿对象的时候我们也不得不对他们进行强制得类型转换。JDK5引入了泛型，它允许指定集合里元素的类型，这样你可以得到强类型在编译时刻进行类型检查的好处。

####2.For-Each循环

For-Each循环得加入简化了集合的遍历。但跟Python、Ruby相比，还是不够优雅。

####3.自动装包/拆包(Autoboxing/unboxing)

自动装包/拆包大大方便了基本类型数据和它们包装类地使用。
自动装包：基本类型自动转为包装类.(int >> Integer)
自动拆包：包装类自动转为基本类型.(Integer >> int)

####4.枚举(Enums)

JDK5引入了一个新关键字enum，我们可以这样来定义一个枚举类型。相信学过C++的对此毫不陌生。

####5.可变参数(Varargs)

可变参数使程序员可以声明一个接受可变数目参数的方法。注意，可变参数必须是函数声明中的最后一个参数。不需要重载了，多cool！

###JDK6

JDK6（代号“野马”）增加了对脚本语言的支持(JSR 223)，原理上是将脚本语言编译成bytecode，这样脚本语言也能享用Java平台的诸多优势，包括可移植性，安全等，另外，由于现在是编译成bytecode后再执行，所以比原来边解释边执行效率要高很多。跟5相比，在性能方面有了不错的提升，但在API库方面的新特性略少。

####1.通用注解（Common Annotations）

Common annotations原本是Java EE 5.0(JSR 244)规范的一部分，现在SUN把它的一部分放到了Java SE 6.0中。为其他相关的Java技术定义一套公共Annotation，避免重复建设，保证Java SE和Java EE各种技术的一致性

####2.对脚本语言的支持 

新引入的对JSR 223的支持，它旨在定义一个统一的规范，使得Java 应用程序可以通过一套固定的接口与各种脚本引擎交互，从而达到在 Java 平台上调用各种脚本语言的目的。

####3.XML API与Web服务 

提供的XML处理框架，以及在此框架之上结合注释（Annotation）技术，提供了强大的针对 Web服务的支持。


###JDK7

JDK7对Java语法有少量更新，重点是在易用性和便捷性的改进。 

####1.二进制变量  

JDK7开始，终于可以用二进制来表示整数(byte,short,int和long)。使用二进制的好处是，可以是代码更容易被理解。语法非常简单，只要在二进制数值前面加 0b或者0B。

####2.数字可以出现下划线  

对于一些比较大的数字，我们定义起来总是不方面，经常缺少或者增加位数。JDK7为我们提供了一种解决方案，下划线可以出现在数字字面量。如：1_000_000_000

####3.switch 语句可以用字符串

我想这个是期待已久的功能了吧，现在才出来，不得不吐槽！

####4.diamond运算符<> 

以你创建一个泛型实例，不需要再详细说明类型，只需用<>,编译器会自动帮你匹配。如：

{% highlight java %}
List<String> list = new ArrayList<>();
{% endhighlight %}

####5.对集合类的语言支持 *实际发布后发现并不支持，系误传*

包含对创建集合类的第一类语言支持。这意味着集合类的创建可以像Ruby和Python那样了。如：

{% highlight java %}
List<String> list = ["item"];  
Map<String,Integer> map = {"key" : 1};  
{% endhighlight %}

####6.自动资源管理 

Java中某些资源是需要手动关闭的，如InputStream，Writes，Sockets等。这个新的语言特性允许try语句本身申请更多的资源，这些资源作用于try代码块，并自动关闭。 

{% highlight java %}
//以前是这样：
BufferedReader br = new BufferedReader(new FileReader(path)); 
try { 
  return br.readLine(); 
} finally { 
  br.close(); 
} 
 
//现在变成了这样： 
try (BufferedReader br = new BufferedReader(new FileReader(path)) { 
  return br.readLine(); 
} 
{% endhighlight %}

####7.捕获多种异常

一个catch可以捕获多个异常，这样可以减少重复代码。每个异常之间用`|`隔开。

{% highlight java %}
try { 
  BufferedReader reader = new BufferedReader(new FileReader("")); 
  Connection con = null; 
  Statement stmt = con.createStatement(); 
} catch (IOException | SQLException e) { 
  //捕获多个异常，e就是final类型的 
  e.printStackTrace(); 
} 
{% endhighlight %}

###JDK8

在未发布之前，java社区也在一直期盼着在JDK8，对我个人而言，最期待莫过于lambda表达式。

####1.lambda表达式

lambda包含3个部分：

* （1）括弧包起来的参数 

* （2）一个箭头 

* （3）方法体，可以是单个语句，也可以是语句块 

看看以前是如何排序的：

{% highlight java %}
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
{% endhighlight %}

再看看现在：

{% highlight java %}
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
{% endhighlight %}

还可以更短：

{% highlight java %}
Collections.sort(names, (String a, String b) -> b.compareTo(a));
{% endhighlight %}

更短...

{% highlight java %}
Collections.sort(names, (a, b) -> b.compareTo(a));
{% endhighlight %}

####2.接口的默认方法

Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可，这个特征又叫做扩展方法，示例如下：

{% highlight java %}
interface Formula {
    double calculate(int a);
    
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
{% endhighlight %}

####3.方法与构造函数引用

Java 8允许你使用`::`关键字来传递方法或者构造函数引用。

###JDK9

本版本一项主要内容就是实现JDK源码的模块化。还将包含三个新的API：

* 针对广泛应用的JSON，加入轻量级JSON API

* 针对现有HttpClient过于抽象、难以使用，新增HTTP 2 Client，将支持HTTP2.0和websocket

* 改进对OS进程控制和管理的Process API
