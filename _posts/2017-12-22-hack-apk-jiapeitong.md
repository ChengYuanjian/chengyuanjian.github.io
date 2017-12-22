---
layout: post
title:  Hack APK&签名回编译
description:
keywords: Java,Android
category: Java
tags: [Java,Android]
---

#### 准备工作

前提是安装好JDK并配置环境变量，然后下载以下反编译工具

* [apktool](https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/osx/apktool)：链接内容另存为apktool.sh
* [apktool.jar](https://bitbucket.org/iBotPeaches/apktool/downloads/?tab=downloads)：选择最新版下载，并重命名为apktool.jar
* [jd-gui](http://jd.benow.ca/)：将jar反编译为java的工具，方便阅读源代码，如果不需要修改java源码，也可以不用下载
* [dex2jar](http://sourceforge.net/projects/dex2jar/files/)：将dex反编译为jar，与上面配合使用

<!-- more -->

#### 反编译过程

##### 1.反编译apk

{% highlight sh %} 
sudo apktool d cyj.apk
{% endhighlight %}
则在当前目录会生成名为`cyj`的文件夹，存放的是反编译后的内容

##### 2.反编译源代码

{% highlight sh %} 
unzip cyj.apk -d src #获取dex文件
sh d2j-dex2jar.sh ../src/classes.dex #将dex文件转为jar
{% endhighlight %}

##### 3.阅读源代码

将上一步生成的classes-dex2jar.jar文件使用jd-gui工具打开，即可看到源代码（一般情况下，都会被混淆）

##### 4.修改smali文件

第三步可以修改java代码来实现hack的效果，但本人更推荐修改smali文件。因为反编译后的java代码跟原始结构还是会存在一些差异。smali文件存放在`cyj/smali`下面，和class文件基本是一一
对应的。可以互相对比来理解代码逻辑。

##### 5.回编译

{% highlight sh %} 
sudo apktool b cyj
{% endhighlight %}
则会在`cyj/dist`下生成新的apk，但此时还不可使用

##### 5.加签名

{% highlight sh %} 
keytool -genkeypair -alias "cheng" -keyalg "RSA" -keystore "cheng.keystore" #生成keystore
sudo jarsigner -verbose -keystore cheng.keystore -signedjar cyj_signed.apk cyj.apk cheng #加签名
{% endhighlight %}
最终生成的`cyj_signed.apk`就是hack后的apk了。
