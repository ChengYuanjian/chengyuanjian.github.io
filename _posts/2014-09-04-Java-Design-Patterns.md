---
layout:         post
title:          Java 设计模式
description:    Java 23中设计模式说明和示例
keywords: Java, Design
category: Java
tags: [Java]
---

设计模式`Design pattern`是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。 毫无疑问，设计模式于己于他人于系统都是多赢的，设计模式使代码编制真正工程化，设计模式是软件工程的基石，如同大厦的一块块砖石一样。项目中合理的运用设计模式可以完美的解决很多问题，每种模式在现在中都有相应的原理来与之对应，每一个模式描述了一个在我们周围不断重复发生的问题，以及该问题的核心解决方案。

*说明：本文资料搜集于网络*

<!-- more -->

------------------

在Java中，有23种设计模式，可分为三大类：

* 1.__创建型Creational Patterns(5)__：抽象工厂`Abstract Factory`，工厂方法`Factory Method`，建造者`Builder`，原型`Prototype`，单例`Singleton`

* 2.__结构型Structural Patterns(7)__：适配器`Adapter`，桥接`Bridge`，复合`Composite`，装饰`Decorator`，外观`Facade`，享元`Flyweight`，代理`Proxy`

* 3.__行为型Behavorial Patterns(11)__：责任链`Chain Of Responsibility`，命令`Command`，解释器`Interpreter`，迭代`Iterator`，中介`Mediator`，备忘`Memento`，观察者`Observer`，状态`State`，策略`Strategy`，模板方法`Template Method`，访问者`Visitor`

此外，还有两类：并发型模式和线程池模式。用一个图片来整体描述一下：

![设计模式之间的关系图](http://dl.iteye.com/upload/attachment/0083/1179/57a92d42-4d84-3aa9-a8b9-63a0b02c2c36.jpg "设计模式之间的关系图【来源于网络】")

-------------------
###创建型

####1.工厂方法|Factory Method

工厂方法模式分为三种：

* 普通工厂模式，就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。如图：

![普通工厂模式](http://dl.iteye.com/upload/attachment/0083/1180/421a1a3f-6777-3bca-85d7-00fc60c1ae8b.png)

* 多个工厂方法模式，是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象。

![多个工厂方法模式](http://dl.iteye.com/upload/attachment/0083/1181/84673ccf-ef89-3774-b5cf-6d2523cd03e5.jpg)

* 静态工厂方法模式，将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。

{% highlight java %}
public class SendFactory {
	
	public static Sender produceMail(){
		return new MailSender();
	}
	
	public static Sender produceSms(){
		return new SmsSender();
	}
}
{% endhighlight %}

总体来说，工厂模式适合：凡是出现了大量的产品需要创建，并且具有共同的接口时，可以通过工厂方法模式进行创建。

---------------------------

####2.抽象工厂|Abstract Factory

工厂方法模式有一个问题就是，类的创建依赖工厂类，也就是说，如果想要拓展程序，必须对工厂类进行修改，这违背了闭包原则，所以，从设计角度考虑，有一定的问题，如何解决？就用到抽象工厂模式，创建多个工厂类，这样一旦需要增加新的功能，直接增加新的工厂类就可以了，不需要修改之前的代码。

![抽象工厂](http://img.my.csdn.net/uploads/201211/29/1354159363_7245.PNG)

这个模式的好处就是，如果你现在想增加一个新功能只需做一个实现类，实现Sender接口，同时做一个工厂类，实现Provider接口，就OK了，无需去改动现成的代码，拓展性较好。

---------------------------

####3.单例模式|Singleton

在Java应用中，单例对象能保证在一个JVM中，该对象只有一个实例存在。

{% highlight java%}
public class SingletonTest {

	private static SingletonTest instance = null;

	private SingletonTest() {
	}

	private static synchronized void syncInit() {
		if (instance == null) {
			instance = new SingletonTest();
		}
	}

	public static SingletonTest getInstance() {
		if (instance == null) {
			syncInit();
		}
		return instance;
	}
}
{% endhighlight %}

---------------------------

####4.建造者模式|Builder

工厂类模式提供的是创建单个类的模式，而建造者模式则是将各种产品集中起来进行管理，用来创建复合对象，所谓复合对象就是指某个类具有不同的属性。

{% highlight java %}
public class Builder {
	
	private List<Sender> list = new ArrayList<Sender>();
	
	public void produceMailSender(int count){
		for(int i=0; i<count; i++){
			list.add(new MailSender());
		}
	}
	
	public void produceSmsSender(int count){
		for(int i=0; i<count; i++){
			list.add(new SmsSender());
		}
	}
}
{% endhighlight %}

可以看出，建造者模式将很多功能集成到一个类里，这个类可以创造出比较复杂的东西。所以与工长模式的区别就是：工厂模式关注的是创建单个产品，而建造者模式则关注创建复合对象。

---------------------------

####5.原型模式|Prototype

原型模式虽然是创建型的模式，但是与工厂模式没有关系。该模式的思想就是将一个对象作为原型，对其进行复制、克隆，产生一个和原对象类似的新对象。在Java中，复制对象是通过clone()实现的。

{% highlight java %}
public class Prototype implements Cloneable {

	public Object clone() throws CloneNotSupportedException {
		Prototype proto = (Prototype) super.clone();
		return proto;
	}
}
{% endhighlight %}

* 浅复制：将一个对象复制后，基本数据类型的变量都会重新创建，而引用类型，指向的还是原对象所指向的。

* 深复制：将一个对象复制后，不论是基本数据类型还有引用类型，都是重新创建的。

{% highlight java %}
public class Prototype implements Cloneable, Serializable {

	private static final long serialVersionUID = 1L;
	private String string;


	/* 浅复制 */
	public Object clone() throws CloneNotSupportedException {
		Prototype proto = (Prototype) super.clone();
		return proto;
	}

	/* 深复制 */
	public Object deepClone() throws IOException, ClassNotFoundException {

		/* 写入当前对象的二进制流 */
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		ObjectOutputStream oos = new ObjectOutputStream(bos);
		oos.writeObject(this);

		/* 读出二进制流产生的新对象 */
		ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
		ObjectInputStream ois = new ObjectInputStream(bis);
		return ois.readObject();
	}

}
{% endhighlight %}

---------------------------
---------------------------
###结构型

对象的适配器模式是其他结构型模式的起源，如图：
![关系图](http://img.my.csdn.net/uploads/201211/29/1354192484_5322.PNG)

####6.适配器模式|Adapter

适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题。主要分为三类：类的适配器模式、对象的适配器模式、接口的适配器模式。

* 类的适配器模式

![类的适配器模式](http://img.my.csdn.net/uploads/201211/29/1354187872_4972.PNG)

核心思想就是：有一个Source类，拥有一个方法，待适配，目标接口时Targetable，通过Adapter类，将Source的功能扩展到Targetable

* 对象的适配器模式

![对象的适配器模式](http://img.my.csdn.net/uploads/201211/29/1354188530_8387.PNG)

基本思路和类的适配器模式相同，只是将Adapter类作修改，这次不继承Source类，而是持有Source类的实例，以达到解决兼容性的问题。

* 接口的适配器模式

有时我们写的一个接口中有多个抽象方法，当我们写该接口的实现类时，必须实现该接口的所有方法，这明显有时比较浪费，因为并不是所有的方法都是我们需要的。借助于一个抽象类，该抽象类实现了该接口，实现了所有的方法，而我们不和原始的接口打交道，只和该抽象类取得联系，所以我们写一个类，继承该抽象类，重写我们需要的方法。

![接口的适配器模式](http://img.my.csdn.net/uploads/201211/29/1354191586_2062.PNG)

* 类的适配器模式：当希望将一个类转换成满足另一个新接口的类时，可以使用类的适配器模式，创建一个新类，继承原有的类，实现新的接口即可。

* 对象的适配器模式：当希望将一个对象转换成满足另一个新接口的对象时，可以创建一个Wrapper类，持有原类的一个实例，在Wrapper类的方法中，调用实例的方法就行。

* 接口的适配器模式：当不希望实现一个接口中所有的方法时，可以创建一个抽象类Wrapper，实现所有方法，我们写别的类的时候，继承抽象类即可。

-----------------------------------

####7.装饰模式|Decorator

装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例，关系图如下：

![装饰模式关系图](http://img.my.csdn.net/uploads/201211/29/1354196367_3730.PNG)

Source类是被装饰类，Decorator类是一个装饰类，可以为Source类动态的添加一些功能。

装饰器模式的应用场景：

* 1、需要扩展一个类的功能。

* 2、动态的为一个对象增加功能，而且还能动态撤销。（继承不能做到这一点，继承的功能是静态的，不能动态增删。）

缺点：产生过多相似的对象，不易排错！

-----------------------------------

####8.代理模式|Proxy

代理模式就是多一个代理类出来，替原对象进行一些操作。

![代理模式关系图](http://img.my.csdn.net/uploads/201211/29/1354197582_1664.PNG)

{% highlight java %}
public class Source implements Sourceable {

	@Override
	public void method() {
		System.out.println("the original method!");
	}
}

public class Proxy implements Sourceable {

	private Source source;
	public Proxy(){
		super();
		this.source = new Source();
	}
	@Override
	public void method() {
		before();
		source.method();
		atfer();
	}
	private void atfer() {
		System.out.println("after proxy!");
	}
	private void before() {
		System.out.println("before proxy!");
	}
}
{% endhighlight %}


-----------------------------

####9.外观模式|Facade

外观模式是为了解决类与类之家的依赖关系的，像spring一样，可以将类和类之间的关系配置到配置文件中，而外观模式就是将他们的关系放在一个Facade类中，降低了类类之间的耦合度。

![外观模式示例图](http://img.my.csdn.net/uploads/201211/29/1354200158_3667.PNG)

如果我们没有Computer类，那么，CPU、Memory、Disk他们之间将会相互持有实例，产生关系，这样会造成严重的依赖，修改一个类，可能会带来其他类的修改，这不是我们想要看到的，有了Computer类，他们之间的关系被放在了Computer类里，这样就起到了解耦的作用，这就是外观模式！

-------------------------------

####10.桥接模式|Bridge

桥接模式就是把事物和其具体实现分开，使他们可以各自独立的变化。桥接的用意是：将抽象化与实现化解耦，使得二者可以独立变化，像我们常用的JDBC桥DriverManager一样，JDBC进行连接数据库的时候，在各个数据库之间进行切换，基本不需要动太多的代码，原因就是JDBC提供统一接口，每个数据库提供各自的实现，用一个叫做数据库驱动的程序来桥接就行了。

![桥接模式示例图](http://img.my.csdn.net/uploads/201211/30/1354242717_1732.PNG)

-------------------------------

####11.组合模式|Composite

组合模式有时又叫部分-整体模式，在处理类似树形结构的问题时比较方便，看看关系图：

![组合模式示例图](http://img.my.csdn.net/uploads/201211/30/1354243465_5968.PNG)

使用场景：将多个对象组合在一起进行操作，常用于表示树形结构中，例如二叉树，数等。

-------------------------------

####12.享元模式|Flyweight

享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。一提到共享池，我们很容易联想到Java里面的JDBC连接池，想想每个连接的特点，我们不难总结出：适用于作共享的一些个对象，他们有一些共有的属性，就拿数据库连接池来说，url、driverClassName、username、password及dbname，这些属性对于每个连接来说都是一样的，所以就适合用享元模式来处理，建一个工厂类，将上述类似属性作为内部数据，其它的作为外部数据，在方法调用时，当做参数传进来，这样就节省了空间，减少了实例的数量。

![享元模式示例图](http://img.my.csdn.net/uploads/201211/30/1354258649_9039.PNG)

通过连接池的管理，实现了数据库连接的共享，不需要每一次都重新创建连接，节省了数据库重新创建的开销，提升了系统的性能！

-------------------------------
-------------------------------

###行为型

这11中模式的关系：
第一类：通过父类与子类的关系进行实现。第二类：两个类之间。第三类：类的状态。第四类：通过中间类

![行为型模式关系图](http://img.my.csdn.net/uploads/201211/30/1354264672_7575.PNG)

####13.策略模式|strategy

策略模式定义了一系列算法，并将每个算法封装起来，使他们可以相互替换，且算法的变化不会影响到使用算法的客户。需要设计一个接口，为一系列实现类提供统一的方法，多个实现类实现该接口，设计一个抽象类（可有可无，属于辅助类），提供辅助函数。

![策略模式关系图](http://img.my.csdn.net/uploads/201211/30/1354268408_3835.PNG)

策略模式的决定权在用户，系统本身提供不同算法的实现，新增或者删除算法，对各种算法做封装。因此，策略模式多用在算法决策系统中，外部用户只需要决定用哪个算法即可。

------------------------------------

####14.模板方法模式|Template Method

一个抽象类中，有一个主方法，再定义1...n个方法，可以是抽象的，也可以是实际的方法，定义一个类，继承该抽象类，重写抽象方法，通过调用抽象类，实现对子类的调用。

![模板方法示例图](http://img.my.csdn.net/uploads/201211/30/1354283441_1858.PNG)

就是在AbstractCalculator类中定义一个主方法calculate，calculate()调用spilt()等，Plus和Minus分别继承AbstractCalculator类，通过对AbstractCalculator的调用实现对子类的调用。

####15.观察者模式|Observer

观察者模式类似于邮件订阅和RSS订阅，当你订阅了该文章，如果后续有更新，会及时通知你。简单来讲就一句话：当一个对象变化时，其它依赖该对象的对象都会收到通知，并且随着变化！对象之间是一种一对多的关系。

![观察者模式示例图](http://img.my.csdn.net/uploads/201211/30/1354285683_8317.PNG)

MySubject类就是我们的主对象，Observer1和Observer2是依赖于MySubject的对象，当MySubject变化时，Observer1和Observer2必然变化。AbstractSubject类中定义着需要监控的对象列表，可以对其进行修改：增加或删除被监控对象，且当MySubject变化时，负责通知在列表内存在的对象。

----------------------------------

####16.迭代模式|Iterator

迭代器模式就是顺序访问聚集中的对象，一般来说，集合中非常常见。这句话包含两层意思：一是需要遍历的对象，即聚集对象，二是迭代器对象，用于对聚集对象进行遍历访问。

![迭代器模式示例图](http://img.my.csdn.net/uploads/201211/30/1354289861_5945.PNG)

MyCollection中定义了集合的一些操作，MyIterator中定义了一系列迭代操作，且持有Collection实例。

----------------------------------

####17.责任链模式|Chain of Responsibility

责任链模式，有多个对象，每个对象持有对下一个对象的引用，这样就会形成一条链，请求在这条链上传递，直到某一对象决定处理该请求。但是发出者并不清楚到底最终哪个对象会处理该请求，所以，责任链模式可以在隐瞒客户端的情况下，对系统进行动态的调整。

{% highlight java %}
public interface Handler {
	public void operator();
}
{% endhighlight %}

{% highlight java %}
public abstract class AbstractHandler {
	
	private Handler handler;

	public Handler getHandler() {
		return handler;
	}

	public void setHandler(Handler handler) {
		this.handler = handler;
	}
	
}
{% endhighlight %}

{% highlight java %}
public class MyHandler extends AbstractHandler implements Handler {

	private String name;

	public MyHandler(String name) {
		this.name = name;
	}

	@Override
	public void operator() {
		System.out.println(name+"deal!");
		if(getHandler()!=null){
			getHandler().operator();
		}
	}
}
{% endhighlight %}

{% highlight java %}
public class Test {

	public static void main(String[] args) {
		MyHandler h1 = new MyHandler("h1");
		MyHandler h2 = new MyHandler("h2");
		MyHandler h3 = new MyHandler("h3");

		h1.setHandler(h2);
		h2.setHandler(h3);

		h1.operator();
	}
}
{% endhighlight %}

链接上的请求可以是一条链，可以是一个树，还可以是一个环，模式本身不约束这个，需要我们自己去实现，同时，在一个时刻，命令只允许由一个对象传给另一个对象，而不允许传给多个对象。

-------------------------------------------

####18.命令模式|Command

命令模式的目的就是达到命令的发出者和执行者之间解耦，实现请求和执行分开。Struts其实就是一种将请求和呈现分离的技术。

![命令模式示例图](http://img.my.csdn.net/uploads/201212/01/1354297121_8890.PNG)

Invoker是调用者，Receiver是被调用者，MyCommand是命令，实现了Command接口，持有接收对象。这个过程好在，三者相互解耦，任何一方都不用去依赖其他人，只需要做好自己的事儿就行，Invoker要的是结果，不会去关注到底Receiver是怎么实现的。

-------------------------------------------

####19.备忘录模式|Memento

主要目的是保存一个对象的某个状态，以便在适当的时候恢复对象。

![备忘录模式示例图](http://img.my.csdn.net/uploads/201212/01/1354329495_6012.PNG)

Original类是原始类，里面有需要保存的属性value及创建一个备忘录类，用来保存value值。Memento类是备忘录类，Storage类是存储备忘录的类，持有Memento类的实例。

--------------------------------------------

####20.状态模式|State

核心思想就是：当对象的状态改变时，同时改变其行为。状态模式就两点：1、可以通过改变状态来获得不同的行为。2、别人能同时看到你的变化。

![状态模式示例图](http://img.my.csdn.net/uploads/201212/01/1354367681_4639.PNG)

--------------------------------------------

####21.访问者模式|Visitor

访问者模式是一种分离对象数据结构与行为的方法，通过这种分离，可达到为一个被访问者动态添加新的操作而无需做其它的修改的效果。。把数据结构和作用于结构上的操作解耦合，使得操作集合可相对自由地演化。

该模式适用场景：如果我们想为一个现有的类增加新功能，不得不考虑几个事情：

* 1、新功能会不会与现有功能出现兼容性问题？

* 2、以后会不会再需要添加？

* 3、如果类不允许修改代码怎么办？

面对这些问题，最好的解决方法就是使用访问者模式。访问者模式适用于数据结构相对稳定算法又易变化的系统。

--------------------------------------------

####22.中介者模式|Mediator

中介者模式也是用来降低类类之间的耦合的，因为如果类类之间有依赖关系的话，不利于功能的拓展和维护，因为只要修改一个对象，其它关联的对象都得进行修改。如果使用中介者模式，只需关心和Mediator类的关系，具体类类之间的关系及调度交给Mediator就行，这有点像spring容器的作用。

![中介模式示例图](http://img.my.csdn.net/uploads/201212/01/1354375826_1485.PNG)


--------------------------------------------

####23.解释器模式|Interpreter

一般主要应用在OOP开发中的编译器的开发中。解释器模式用来做各种各样的解释器，如正则表达式等的解释器等。

![解释器模式示例图](http://img.my.csdn.net/uploads/201212/02/1354377970_2591.PNG)

Context类是一个上下文环境类，Plus和Minus分别是用来计算的实现。

----------------------------------------------

* [Java设计模式源码示例](http://www.fluffycat.com/Java-Design-Patterns/)
