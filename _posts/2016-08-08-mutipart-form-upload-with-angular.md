---
layout: post
title:  Multipart/form-data File Upload with AngularJS
description: 
keywords: JavaScript,Angular
category: JavaScript
tags: [JavaScript,Angular]
---

近期做一个新项目，基于CloudFoundry的PaaS管理平台，前端技术选定用AngularJS。由于人手紧缺，虽然我很久没接触前端了，但心想着了解一下也不错，便开始着手写AngularJS。
双向数据绑定的特性确实很强大，看了一下官方的Demo，很快即可上手。前期普通的CRUD页面都很顺利，直到遇到文件上传，CF的API：[查看CF上传接口](https://apidocs.cloudfoundry.org/193/apps/uploads_the_bits_for_an_app.html)

看起来很简单，对不对？就一普通的Resful Service而已。然而却耗费了我大量的时间。

<!-- more -->

#### 上传组件 

Angular拥有很多开源的组件，上传常用的有[angular-file-upload](https://github.com/nervgh/angular-file-upload)、[ng-flow](https://github.com/flowjs/ng-flow)
等，用法可以参考相应的示例。

#### 问题的症结

入参有普通的文本，也有文件；即意味着`content-type`不能是`application/json`。Angular对`POST`和`PUT`请求，默认的`content-type`就是`application/json`。
有人就说了，那手动设置为`multipart/form-data`可以吗？答案是不可以。

当指定了表单类型为`multipart/form-data`，上传文件不在使用原有的http协议了。请求内容不再可能以`k = V`方式发送了,而使用了 

分隔符 
内容1
分隔符号 
内容2 

而boundary就是指定分隔符号的标志。 
请求的内容就应该是这样的了:

```
------WebKitFormBoundaryuwYcfA2AIgxqIxA0 
Content-Disposition: form-data; name="body" 
123 
------WebKitFormBoundaryuwYcfA2AIgxqIxA0 
Content-Disposition: form-data; name="name"; filename="11.txt" 
Content-Type: application/x-msdownload 
```

那我们手动加上boundary可以吗？`multipart/form-data; boundary=ABCD`，这种方式通过后台语言调用可以这样做，例如Java（注意两处的注释）：

{% highlight java %}
CloseableHttpClient client = HttpClients.createDefault();

HttpPut request = new HttpPut(url);

request.setHeader("Authorization", authorization);
request.setHeader("Content-Type", "multipart/form-data;boundary="+boundary);//设置1

MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setBoundary(boundary);//设置2

File file = new File("/Users/CYJ/Downloads/application.zip");
if (file.exists()) {
	builder.addTextBody("resources", "[]");
	builder.addBinaryBody("application", file);

	request.setEntity(builder.build());
	
	HttpResponse httpResponse = client.execute(request);
	HttpEntity httpEntity = httpResponse.getEntity();
	String content = EntityUtils.toString(httpEntity);

	System.out.println(content);
}
{% endhighlight %}

但通过Angular这种前端请求的方式，却不奏效，除非对请求报文人为插入`boundary`，[手动插入boundary示例](http://woxiangbo.iteye.com/blog/1751740)。是不是巨麻烦？
而且我们都知道JS处理文件效率较低，这种工作还是交给后台比较合适。

#### 解决方案

说了那么多，应该怎么做呢？其实很简单：

{% highlight js %}
var config = {
	transformRequest: angular.identity,
	headers: {'Content-Type': undefined}
};
{% endhighlight %}

首先指定`content-type`为`undefined`，用来覆盖angular默认方式。其次，设置`transformRequest`为`angular.identity`。因为angular默认的`transformRequest`
会将数据进行序列化，通过这样设置可以保存原始数据。从而浏览器可以自动识别`content-type`并填充`boundary`。
于是乎，问题解决。
