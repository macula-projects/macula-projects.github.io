---
type: docs
title: "统一动态属性配置"
linkTitle: "统一动态属性配置"
weight: 2
description: "本文介绍了Macula的统一动态属性配置方式"
---

如何使用：
 
 ### 在applicationContext-root.xml增加Zookeeper连接信息，如：
```xml
<bean id="maculaCuratorFramework" class="org.macula.core.configuration.reloadable.CuratorFrameworkFactoryBean" init-method="start" destroy-method="stop">
    <constructor-arg index="0" type="java.lang.String">
        <value>172.20.70.21:2181</value> <!-- Zookeeper 集群地址-->
    </constructor-arg>
    <constructor-arg index="1" type="org.apache.curator.RetryPolicy"> <!-- 连接重试策略 -->
        <ref bean="maculaCuratorRetryPolicy"/>
    </constructor-arg>
</bean>	

<bean id="maculaCuratorRetryPolicy" class="org.apache.curator.retry.ExponentialBackoffRetry">
    <constructor-arg index="0">
        <value>1000</value>
    </constructor-arg>
    <constructor-arg index="1">
        <value>3</value>
    </constructor-arg>
</bean>
```

### 通过后台管理界面配置动态属性

动态属性配置分为应用分组级别和应用级别，不同级别的动态属性可见范围不同

![](/images/chapter2/dynaconfig01.png)

### 代码中使用

程序里有两种方式可以使用这里配置的属性。
#### 1) 通过Configuration.getProperty()的方式，如：
```java
Configuration.getProperty("MyName");
```

#### 2) 使用@Value注解，如：
```java
@Value("${MyName}")
private String myName;
```

### 优先级

动态属性配置里的属性拥有最高优先级（也就是最后加载），会覆盖macual.properties和环境变量中的同名属性。优先级顺序是：动态属性配置 > 环境变量 > macual.properties

### 动态刷新

在后台管理页面中，属性编辑保存后，不需重启应用实例，属性会自动刷新。
- 对于使用Configuration.getProperty()这种方式的，更新后调用Configuration.getProperty()得到的便是刷新后的值；
- 对于使用@Value的方式，要按以下步骤（当然并不是所有的属性都适合动态刷新，所以你要清楚自己在做什么）：
1. 首先该属性所在的类要实现PropertiesReloadable接口或继承AbstractPropertiesReloadable，例如：
```java
public class Abc implements PropertiesReloadable {
    @Value("${MyName}")
    @Reloadable
    private String myName;

    public void setMyName(String myName) {
        this.myName = myName;
    }
    ...
}
```
PropertiesReloadable接口有以下两个方法：
```java
public interface PropertiesReloadable {
    /**
     * 自定义属性reload前采取的动作
     */
    void beforePropertiesReloaded();
    
    /**
     * 自定义属性reload后采取的动作
     */
    void afterPropertiesReloaded();
}
```
这两个方法提供机会给类实例在属性刷新前、刷新后做一些自定义的处理工作，框架在刷新属性时会自动调用这两个方法。AbstractPropertiesReloadable提供了这两个方法的空实现。

2. 在该属性上添加@org.macula.core.configuration.reloadable.Reloadable注解

3. 给该属性添加setter方法。不要忘了这一步，不然属性就刷新不了。
