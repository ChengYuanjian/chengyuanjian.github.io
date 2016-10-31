---
layout: post
title:  Spring事务管理
description: 
keywords: Spring
category: Spring
tags: [Spring,Transaction]
---

### 事务管理器

事务分物理事务和逻辑事务；物理事务是数据库提供的事务支持，如JDBC、JTA等，Spring中一般指逻辑事务，它提供了多种事务管理器，常用的有：

* DataSourceTransactionManager

位于org.springframework.jdbc.datasource包中，数据源事务管理器，提供对单个javax.sql.DataSource事务管理，用于Spring JDBC抽象框架、iBatis框架的事务管理；

* HibernateTransactionManager

位于org.springframework.orm.hibernate3或者hibernate4包中，提供对单个org.hibernate.SessionFactory事务支持，用于集成Hibernate框架时的事务管理；该事务管理器只支持Hibernate3+版本，且Spring3.0+版本只支持Hibernate 3.2+版本；

* JtaTransactionManager

位于org.springframework.transaction.jta包中，实现Java原生事务管理，提供对分布式事务管理的支持(多数据源)，并将事务管理委托给Java EE应用服务器事务管理器；

* JpaTransactionManager

位于org.springframework.orm.jpa包中，提供对单个javax.persistence.EntityManagerFactory事务支持，用于集成JPA实现框架时的事务管理；

<!-- more -->

### 事务传播行为

Spring管理的事务是逻辑事务，定义了七种传播行为（propagation behavior）：

|传播行为|	含义|
|---------------------|---------------------|
|PROPAGATION_REQUIRED|	必须有逻辑事务，否则新建一个事务。如果当前存在一个逻辑事务，则加入该逻辑事务，否则将新建一个逻辑事务|
|PROPAGATION_SUPPORTS|	如果存在事务的话，那么该方法会在这个事务中运行，否则就以非事务方式运行|
|PROPAGATION_MANDATORY|	表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常|
|PROPAGATION_REQUIRED_NEW| 表示总会为自己创建一个新的事务。如果已存在事务，在该方法运行期间，当前事务将被挂起，直到自己的事务提交后才会恢复执行。|
|PROPAGATION_NOT_SUPPORTED|	指定当前方法以非事务方式执行操作，如果当前存在事务，就把当前事务挂起，等我以非事务的状态运行完，再继续原来的事务。|
|PROPAGATION_NEVER| 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常|
|PROPAGATION_NESTED| 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的|

### Spring实现事务管理

Spring提供了编程式和声明式事务管理。编程式事务运行在代码中精确定义事务边界，而声明式可以将操作和事物规则解耦。

#### 编程式事务

Spring提供两种方式的编程式事务管理，分别是：使用TransactionTemplate和直接使用PlatformTransactionManager。

##### 使用TransactionTemplate

采用TransactionTemplate和采用其他Spring模板，如JdbcTempalte和HibernateTemplate是一样。它使用回调，把应用程序从处理取得和释放资源中解脱出来。TransactionTemplate是线程安全的。代码片段：

```java
TransactionTemplate tt = new TransactionTemplate(); // 新建一个TransactionTemplate
tt.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED); 
tt.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);//指定传播行为，其默认为PROPAGATION_REQUIRED
    Object result = tt.execute(
        new TransactionCallback(){  //如果不需要返回值，可使用TransactionCallbackWithoutResult
            public Object doInTransaction(TransactionStatus status){  
                //TODO 数据库操作
            }  
    }); 
```

##### 使用PlatformTransactionManager

PlatformTransactionManager是一个接口，通常使用其实现类DataSourceTransactionManager：

```java
    DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(); 
    dataSourceTransactionManager.setDataSource(this.getJdbcTemplate().getDataSource()); 
    DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); 
    transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED);  
    TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef); 
    try {
        //TODO 数据库操作
        dataSourceTransactionManager.commit(status);// 提交
    } catch (Exception e) {
        dataSourceTransactionManager.rollback(status);// 回滚
    }
```

#### 声明式事务

声明式事务管理简单、无侵入，对业务代码实现基本无影响。实现的方式也有多种：

##### bean配置

```xml
<!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <bean id="transactionBase" 
            class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean" 
            lazy-init="true" abstract="true"> 
        <!-- 配置事务管理器 --> 
        <property name="transactionManager" ref="transactionManager" /> 
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop> 
            </props> 
        </property> 
    </bean>   

    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.cyj.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <bean id="userDao" parent="transactionBase" > 
        <property name="target" ref="userDaoTarget" />  
    </bean>
```

##### 拦截器配置

```xml
<!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean> 

    <bean id="transactionInterceptor" 
        class="org.springframework.transaction.interceptor.TransactionInterceptor"> 
        <property name="transactionManager" ref="transactionManager" /> 
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop> 
            </props> 
        </property> 
    </bean>

    <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator"> 
        <property name="beanNames"> 
            <list> 
                <value>*Dao</value>
            </list> 
        </property> 
        <property name="interceptorNames"> 
            <list> 
                <value>transactionInterceptor</value> 
            </list> 
        </property> 
    </bean> 

    <!-- 配置DAO -->
    <bean id="userDao" class="com.cyj.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
</beans>
```

##### 基于aop

```xml
<!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="interceptorPointCuts"
            expression="execution(* com.cyj.dao.*.*(..))" />
        <aop:advisor advice-ref="txAdvice"
            pointcut-ref="interceptorPointCuts" />       
    </aop:config>     
```

##### 注解

```xml
 <tx:annotation-driven transaction-manager="transactionManager"/>
 
 <!-- 定义事务管理器（声明式的事务） --> 
 <bean id="transactionManager"
     class="org.springframework.orm.hibernate3.HibernateTransactionManager">
      <property name="sessionFactory" ref="sessionFactory" />
 </bean>
```
 
 此时在DAO上需加上`@Transactional`注解:
 
```java
@Transactional
@Component("userDao")
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {

    public List<User> listUsers() {
        return this.getSession().createQuery("from User").list();
    }  
}
```
