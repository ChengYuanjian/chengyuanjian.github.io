---
layout:         post
title:          Java编程细节之Annotation
description: 
keywords: Java, Annotation
category: Java
tags: [Java,Annotation]
---

`Annotation`注解是JDK5.0及以后版本引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。注解是以`@注解名`在代码中存在的，根据注解参数的个数，我们可以将注解分为：标记注解、单值注解、完整注解三类。它们都不会直接影响到程序的语义，只是作为注解（标识）存在，我们可以通过反射机制编程实现对这些元数据（用来描述数据的数据）的访问。

<!-- more -->

###注解的作用

大致可分为三类：

* 编写文档：通过代码里标识的注解生成文档。

* 代码分析：通过代码里标识的注解对代码进行分析。

* 编译检查：通过代码里标识的注解让编译器能实现基本的编译检查。

-----------------------

###注解的高级应用

####限制注解的使用范围

用`@Target`指定ElementType属性

{% highlight java %}
package java.lang.annotation;
public enum ElementType{
  TYPE,//用于类，接口，枚举但不能是注解
  FIELD,//字段上，包括枚举值
  METHOD,//方法，不包括构造方法
  PARAMETER,//方法的参数
  CONSTRUCTOR,//构造方法
  LOCAL_VARIABLE,//本地变量或catch语句
  ANNOTATION_TYPE,//注解类型(无数据)
  PACKAGE//Java包
}
{% endhighlight %}

示例：
{% highlight java %}
@Target({ElementType.METHOD,ElementType.CONSTRUCTOR})
public @interface MyAnnotation{

}
{% endhighlight %}

####保持性策略

用`@Retention`指定RetentionPolicy属性

{% highlight java %}
packagejava.lang.annotation;
public enum RetentionPolicy{
  SOURCE,//此类型会被编译器丢弃
  CLASS,//此类型注解会保留在class文件中，但JVM会忽略它
  RUNTIME//此类型注解会保留在class文件中，JVM会读取它
}
{% endhighlight %}

示例：
{% highlight java %}
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation{

}
{% endhighlight %}

####文档化功能

Java提供的Documented元注解跟Javadoc的作用是差不多的，其实它存在的好处是开发人员可以定制Javadoc不支持的文档属性，并在开发中应用。

示例：
{% highlight java %}
//让它定制文档化功能
//使用此注解时必须设置RetentionPolicy为RUNTIME
@Documented
public @interface MyAnnotation{

}
{% endhighlight %}

####标注继承

{% highlight java %}
//让它允许继承，可作用到子类
@Inherited
public @interface MyAnnotation{

}
{% endhighlight %}

--------------------------

###自定义注解示例

####自定义注解MyAnnotation

{% highlight java %}
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD,ElementType.CONSTRUCTOR})
public @interface MyAnnotation{
  String name() default "cyj";//给出默认值
  int age() default 0;
}
{% endhighlight %}

####使用自定义注解

{% highlight java %}
public class MyClass{
  @MyAnnotation(name="abc",age=10)
  public void function1(String name, int age){// 使用注解并传入参数的方法
    System.out.println(name + "'s age is " + age);
  } 
  
  @MyAnnotation
  public void function2(String name, int age){// 使用注解并使用默认参数的方法
    System.out.println(name + "'s age is " + age);
  }
}
{% endhighlight %}

####读取注解

**使用反射去读取注解，必须将Retention的值选为Runtime**

{% highlight java %}

Class c=Class.forName("com.xxx.MyClass");
//获取该类所有声明的方法
Method[] methods=c.getDeclaredMethods();

//遍历所有的方法得到各方法上面的注解信息
for(Method method:methods){
  //获取每个方法上面所声明的所有注解信息
  Annotation[] annotations=method.getDeclaredAnnotations();
  
  //获取指定的注解
  MyAnnotation ann=method.getAnnotation(MyAnnotation.class);
  
  //调用方法
  method.invoke(new MyClass(),ann.name(),ann.age());//使用注解传递的值
  
}
{% endhighlight %}

