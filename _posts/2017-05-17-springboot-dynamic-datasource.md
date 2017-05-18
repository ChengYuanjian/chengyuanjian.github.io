---
layout: post
title:  Spring Boot动态多数据源
description:
keywords: Spring
category: Spring
tags: [Spring]
---

在某些特殊场景下，需要访问不同的数据库，把数据捏合起来展示到前端。Spring Boot实现多数据源的方式可谓是多种多样，今天来分享一种较为简单并且无侵害的方式，基本原理是基于AOP和注解。

<!-- more -->

### 1.新增数据源配置

Spring默认的数据源配置为：
{% highlight property %}
spring.datasource.url=""
spring.datasource.username=""
spring.datasource.password=""
spring.datasource.driver-class-name=""
{% endhighlight %}

新增数据源配置(以两个为例，以英文逗号`,`分割)：
{% highlight property %}
spring.custom.datasource.names=ds1,ds2
spring.custom.datasource.ds1.driver-class-name=""
spring.custom.datasource.ds1.url=""
spring.custom.datasource.ds1.username=""
spring.custom.datasource.ds1.password=""

spring.custom.datasource.ds2.driver-class-name=""
spring.custom.datasource.ds2.url=""
spring.custom.datasource.ds2.username=""
spring.custom.datasource.ds2.password=""
{% endhighlight %}

### 2.新增注解

{% highlight java %}
@Target({ElementType.METHOD,ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TargetDataSource {
    String name();
}
{% endhighlight %}

### 3.新增切面处理类

主要用来处理注解前后切换数据源和恢复默认数据
{% highlight java %}
@Slf4j
@Aspect
@Order(-1)
@Component
public class DynamicDataSourceAspect {

    @Before("@annotation(ds)")
    public void changeDataSource(JoinPoint joinPoint, TargetDataSource ds) throws Exception {
        String dsId = ds.name();
        if (DynamicDataSourceContextHolder.containsDataSource(dsId)) {
            log.info("使用数据源[{}]", ds.name());
            DynamicDataSourceContextHolder.setDataSourceType(ds.name());
        } else {
            log.error("数据源[{}]不存在，使用默认数据源[{}]", ds.name(), joinPoint.getSignature());
        }
    }

    @After("@annotation(ds)")
    public void restoreDataSource(JoinPoint joinPoint, TargetDataSource ds) {
        log.info("恢复数据源[{}]->[{}]", ds.name(), joinPoint.getSignature());
        DynamicDataSourceContextHolder.clearDataSourceType();
    }
}
{% endhighlight %}

### 4.处理数据源线程独享

{% highlight java %}
public class DynamicDataSourceContextHolder {

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
    private static List<String> dataSourceIds = Lists.newArrayList();

    public static void setDataSourceType(String dataSourceType) {
        contextHolder.set(dataSourceType);
    }

    public static String getDataSourceType() {
        return contextHolder.get();
    }

    public static void clearDataSourceType() {
        contextHolder.remove();
    }

    public static boolean containsDataSource(String dsId) {
        return dataSourceIds.contains(dsId);
    }

    public static void addDataSourceId(String dsId){
        dataSourceIds.add(dsId);
    }
}
{% endhighlight %}

### 5.动态数据源

{% highlight java %}
public class DynamicDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getDataSourceType();
    }
}
{% endhighlight %}


### 6.具体实现类

根据环境变量生成多数据源，并注册为bean，交给spring容器来管理。

{% highlight java %}
@Slf4j
@Component
public class DynamicDataSourceRegister implements ImportBeanDefinitionRegistrar, EnvironmentAware {

    private ConversionService conversionService = new DefaultConversionService();
    private PropertyValues dataSourcePropertyValues;
    private DataSource defaultDataSource;
    private Map<String, DataSource> customDataSources = Maps.newHashMap();

    private static String DATASOURCE_TYPE_DEFAULT = "org.apache.tomcat.jdbc.pool.DataSource";

    @Override
    public void setEnvironment(Environment environment) {
        initDefaultDataSource(environment);
        initCustomDataSources(environment);
    }

    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        Map<Object, Object> targetDataSources = Maps.newHashMap();
        targetDataSources.put("dataSource", defaultDataSource);
        DynamicDataSourceContextHolder.addDataSourceId("dataSource");

        targetDataSources.putAll(customDataSources);

        customDataSources.keySet().forEach(t -> DynamicDataSourceContextHolder.addDataSourceId(t));

        GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
        genericBeanDefinition.setBeanClass(DynamicDataSource.class);
        genericBeanDefinition.setSynthetic(true);

        MutablePropertyValues mutablePropertyValues = genericBeanDefinition.getPropertyValues();
        mutablePropertyValues.add("defaultTargetDataSource", defaultDataSource);
        mutablePropertyValues.add("targetDataSources", targetDataSources);

        beanDefinitionRegistry.registerBeanDefinition("dataSource", genericBeanDefinition);
    }

    /**
     * 初始化自定义数据源，注意配置的key
     * @param environment
     */
    private void initCustomDataSources(Environment environment) {
        RelaxedPropertyResolver propertyResolver = new RelaxedPropertyResolver(environment, "spring.custom.datasource.");
        for (String dsPrefix : propertyResolver.getProperty("names").split(",")) {
            Map<String, Object> dsMap = propertyResolver.getSubProperties(dsPrefix + ".");
            DataSource ds = buildDataSource(dsMap);
            customDataSources.put(dsPrefix, ds);
            dataBinder(ds, environment);
        }
    }

    /**
     * 初始化默认数据源，即标准的spring配置数据源
     * @param environment
     */
    private void initDefaultDataSource(Environment environment) {
        RelaxedPropertyResolver propertyResolver = new RelaxedPropertyResolver(environment, "spring.datasource.");
        Map<String, Object> dsMap = Maps.newHashMap();
        dsMap.put("type", propertyResolver.getProperty("type"));
        dsMap.put("driver-class-name", propertyResolver.getProperty("driver-class-name"));
        dsMap.put("url", propertyResolver.getProperty("url"));
        dsMap.put("username", propertyResolver.getProperty("username"));
        dsMap.put("password", propertyResolver.getProperty("password"));

        defaultDataSource = buildDataSource(dsMap);

        dataBinder(defaultDataSource, environment);
    }

    /**
     * 根据环境变量绑定数据源
     * @param dataSource
     * @param environment
     */
    private void dataBinder(DataSource dataSource, Environment environment) {
        RelaxedDataBinder dataBinder = new RelaxedDataBinder(dataSource);
        dataBinder.setConversionService(conversionService);
        dataBinder.setIgnoreNestedProperties(false);
        dataBinder.setIgnoreInvalidFields(false);
        dataBinder.setIgnoreUnknownFields(true);

        if (dataSourcePropertyValues == null) {
            Map<String, Object> tmpMap = new RelaxedPropertyResolver(environment, "spring.datasource").getSubProperties(".");
            Map<String, Object> propertyValues = Maps.newHashMap(tmpMap);
            propertyValues.remove("type");
            propertyValues.remove("driver-class-name");
            propertyValues.remove("url");
            propertyValues.remove("username");
            propertyValues.remove("password");
            dataSourcePropertyValues = new MutablePropertyValues(propertyValues);
        }

        dataBinder.bind(dataSourcePropertyValues);
    }

    /**
     * 创建数据源
     * @param dsMap
     * @return
     */
    private DataSource buildDataSource(Map<String, Object> dsMap) {
        try {
            Object type = dsMap.get("type");
            if (null == type) {
                type = DATASOURCE_TYPE_DEFAULT;
            }

            Class<? extends DataSource> dataSourceType = (Class<? extends DataSource>) Class.forName(type.toString());
            String driverClassName = dsMap.get("driver-class-name").toString();
            String url = dsMap.get("url").toString();
            String username = dsMap.get("username").toString();
            String password = dsMap.get("password").toString();

            DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create().driverClassName(driverClassName).url(url).username(username).password(password).type(dataSourceType);
            return dataSourceBuilder.build();
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        return null;
    }
}
{% endhighlight %}


### 7.启动时注册
在启动类引入，完成数据源注册

{% highlight java %}
@Import({DynamicDataSourceRegister.class})
public class App {

}
{% endhighlight %}


### 8.调用示例

在service中加入注解，指明该方法用哪个数据源；如果不加注解，则使用默认数据源，即spring标准配置的数据源。

{% highlight java %}
 @TargetDataSource(name="wind")
 public List<Map> getSample(){
     return sampleMapper.getSample();
 }
{% endhighlight %}
