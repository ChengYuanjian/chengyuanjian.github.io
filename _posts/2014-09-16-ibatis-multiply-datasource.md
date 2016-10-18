---
layout:         post
title:         关于iBatis多数据源、通配符配置
description: 
keywords: Java, iBatis, MyBatis
category: Java
tags: [Java,iBatis,MyBatis]
---

iBatis是一个基于Java的持久层框架，2010年这个项目由apache software foundation迁移到了google code，并且改名为MyBatis。相比Hibernate，iBatis支持自定义SQL的灵活配置，深深吸引了我。同时另一著名的开源框架Spring对iBatis也提供了强大的技术支持，二者完美的结合可以大幅度提高开发效率。在实际的运用中，如果不想依赖spring框架，又想使用iBatis，在某些场景下，有一定的缺陷，如多数据源、通配符等。本文以iBatis2.3版本为例，给出解决思路，仅供参考。

<!-- more -->

####1.iBatis多数据源的支持

采用spring的bean注入的方式，可以很方便地支持多数据源。以下两种方式可以在脱离spring实现同样的效果。

* 方式一，如果使用iBatis进行数据源管理

（1）iBatis提供了数据源的配置，如下：

{% highlight xml %}
  <transactionManager type="JDBC"><!-- 定义了ibatis的事务管理器有3中（JDBC,JTA:容器提供的JTA服务,EXTERNAL:外部事务管理） -->
		<dataSource type="SIMPLE"><!-- type属性指定了数据源的链接类型，也有3种类型(SIMPLE:ibatis内置的dataSource实现,DBCP:基于Apache DBCP连接池组件实现的DataSource封装,JNDI:使用J2EE容器提供的DataSource实现) -->
			<property name="JDBC.Driver"
				value="oracle.jdbc.driver.OracleDriver" />
			<property name="JDBC.ConnectionURL"
				value="jdbc:oracle:thin:@host:port:sid" />
			<property name="JDBC.Username" value="usr" />
			<property name="JDBC.Password" value="pwd" />
			<property name="Pool.MaximumActiveConnections" value="10" />
			<property name="Pool.MaximumIdleConnections" value="5" />
			<property name="Pool.MaximumCheckoutTime" value="120000" />
			<property name="TimeToWait" value="500" />
		</dataSource>
	</transactionManager>
{% endhighlight %}

（2）通过以上方式配置好不同的数据源，并保存为<pre><code>SqlMapConfig\_DS1.xml，SqlMapConfig\_DS2.xml，SqlMapConfig\_DS3.xml</code></pre>。

（3）读取不同的配置文件构造不同的SqlMapClientImpl。
{% highlight java %}
public SqlMapClientImpl getSqlMapClient(String dataSource){
 Reader reader = null;
  try{
    reader = Resources.getResourceAsReader("SqlMapConfig_"+dataSource+".xml");
    return (SqlMapClientImpl)SqlMapClientBuilder.buildSqlMapClient(reader);			
  }catch(IOException e){
    e.printStackTrace();
  }finally{
    if(reader != null)
      reader.close();
  }
}
{% endhighlight %}

（4）拿到SqlMapClientImpl实例后，做CRUD操作即可。

* 方式二，由外部传入Connection

即不用iBatis进行数据源配置，去掉上述的xml配置，自定义Connection传给iBatis使用。一句话即可搞定。
{% highlight java %}
sqlMapClient.setUserConnection(connection);
{% endhighlight %}

这种用法Connection的事务管理、资源分配需要我们自己写额外的代码去维护。

----------------------------

####2.iBatis通配符配置

一般情况下，我们会把sql按照不同的模块配置在不同的xml里。传统的方式，我们需要不停地修改SqlMapConfig.xml。如果可以支持通配符，那就一劳永逸了。同样，spring提供了这样的配置：
{% highlight xml %}
<bean id="sqlMapClient"  
    class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">  
    <property name="dataSource">  
        <ref bean="datasource" />  
    </property>  
    <property name="configLocation" value="classpath:com/sqlmap/sqlmap-config.xml" >  
    </property> 
     <property name="mappingLocations">
        <value>classpath:com/cyj/**/*.xml</value>
    </property>
</bean> 
{% endhighlight %}

脱离了spring，iBatis自身是不支持通配符的，那我们要做的就是更改`com.ibatis.sqlmap.engine.builder.xml.SqlMapConfigParser`的`addSqlMapNodelets()`方法。以下是iBatis原有代码：
{% highlight java %}
parser.addNodelet("/sqlMapConfig/sqlMap", new Nodelet() {
      public void process(Node node) throws Exception {
        vars.errorCtx.setActivity("loading the SQL Map resource");

        Properties attributes = NodeletUtils.parseAttributes(node, vars.properties);

        String resource = attributes.getProperty("resource");
        String url = attributes.getProperty("url");

        if (usingStreams) {
          InputStream inputStream = null;
          if (resource != null) {
            vars.errorCtx.setResource(resource);
            inputStream = Resources.getResourceAsStream(resource);
          } else if (url != null) {
            vars.errorCtx.setResource(url);
            inputStream = Resources.getUrlAsStream(url);
          } else {
            throw new SqlMapException("The <sqlMap> element requires either a resource or a url attribute.");
          }

          if (vars.sqlMapConv != null) {
            inputStream = vars.sqlMapConv.convertXml(inputStream);
          }
          new SqlMapParser(vars).parse(inputStream);
        } else {
          Reader reader = null;
          if (resource != null) {
            vars.errorCtx.setResource(resource);
            reader = Resources.getResourceAsReader(resource);
          } else if (url != null) {
            vars.errorCtx.setResource(url);
            reader = Resources.getUrlAsReader(url);
          } else {
            throw new SqlMapException("The <sqlMap> element requires either a resource or a url attribute.");
          }

          if (vars.sqlMapConv != null) {
            reader = vars.sqlMapConv.convertXml(reader);
          }
          new SqlMapParser(vars).parse(reader);
        }
      }
    });
{% endhighlight %}
改动如下：
{% highlight java %}
 protected void addSqlMapNodelets() {
    parser.addNodelet("/sqlMapConfig/sqlMap", new Nodelet() {
      public void process(Node node) throws Exception {
        vars.errorCtx.setActivity("loading the SQL Map resource");

        Properties attributes = NodeletUtils.parseAttributes(node, vars.properties);

        String resource = attributes.getProperty("resource");
        System.out.println(resource);
        String url = attributes.getProperty("url");

       if(resource.indexOf("*")>0){   
  	     List<String> fileList = getAllResource(resource); //如果有通配符配置，自定义方法getAllResource获取配置资源
  	     /**
 	      * 没有通配符，则走原路线，把原有后半部分代码提取为方法addSqlMapConfigFiles
 	      */
 	     if(fileList==null || fileList.size()<1){
 	    	addSqlMapConfigFiles(resource, url);
 	     }else{
 	    	 for(int i=0;i<fileList.size();i++){   
 	    	      addSqlMapConfigFiles(fileList.get(i), url);   
 	    	   }   
 	     }
 	    }else{   
  	     addSqlMapConfigFiles(resource, url);
  	    }  
      }
   }
  );
  }
{% endhighlight %}
把原有后半部分代码提取为方法addSqlMapConfigFiles：
{% highlight java %}
   protected void addSqlMapConfigFiles(String resource,String url)throws Exception{
   if(usingStreams){
	   InputStream inputStream = null;
	   if (resource != null) {
		   vars.errorCtx.setResource(resource);
	       inputStream = Resources.getResourceAsStream(resource);
	   }else if (url != null) {
		   vars.errorCtx.setResource(url);
	       inputStream = Resources.getUrlAsStream(url);
	   }else{
	    throw new SqlMapException("The <sqlMap> element requires either a resource or a url attribute.");
	   }
	   new SqlMapParser(vars).parse(inputStream);
  }else{
	   Reader reader = null;
	   if (resource != null){
		   vars.errorCtx.setResource(resource);
	       reader = Resources.getResourceAsReader(resource);
	   }else if(url != null){
		   vars.errorCtx.setResource(url);
	       reader = Resources.getUrlAsReader(url);
	   }else{
	    throw new SqlMapException("The <sqlMap> element requires either a resource or a url attribute."); 
	   }
	   new SqlMapParser(vars).parse(reader);
  }
 }
{% endhighlight %}
如果有通配符配置，自定义getAllResource方法，通过getFilePath递归得到所有匹配到的xml资源文件：
{% highlight java %}
protected static List<String> getAllResource(String resource) {
		String dirString = resource.substring(0, resource.indexOf("*"));
		List<String> fileList = new ArrayList<String>();
		SqlMapConfigParser.getFilePath(dirString, fileList);
		return fileList;
	}

	protected static void getFilePath(String resourcePath, List<String> fileList) {
		File file = new File(resourcePath);
		File[] listFiles = file.listFiles();
		if (listFiles != null ) {
			for (int i = 0; i < listFiles.length; i++) {
				if (listFiles[i].isDirectory()) {
					getFilePath(listFiles[i].getPath(), fileList);
				} else {
					String fileNameString = listFiles[i].getPath();
					if (fileNameString.endsWith(".ibatis.xml")) {//这里约定所有以.ibatis.xml结尾的文件为ibatis配置文件
						fileList.add(fileNameString.substring(4,fileNameString.length()));
					}
				}
			}
		}
	}
{% endhighlight %}

由此可以看出，spring对iBatis的良好支持，已经为我们做了大量的封装工作，但也带来额外的bean配置，看个人取舍了。
