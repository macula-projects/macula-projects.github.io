---
type: docs
title: "系统监控"
linkTitle: "系统监控"
weight: 9
description: "本文介绍怎么引入CAT作为系统监控"
---

### 应用监控

Macula是用大众点评开源的[CAT](https://github.com/dianping/cat)作为应用监控的服务器端，并通过插件集成，具体开启应用监控的步骤如下：

#### 首先：如果你是WEB应用需要依赖macula-plugins-catex插件，如果是Dubbo应用，需要依赖macula-plugins-cat插件



#### 创建client.xml

\/data\/appdatas\/cat\/目录下，新建一个client.xml文件\(线上环境是OP配置\)
如果系统是windows环境，则在eclipse运行的盘，比如D盘，新建\/data\/appdatas\/cat\/目录，新建client.xml文件
\/data\/appdatas\/cat\/client.xml,此文件有OP控制,这里的Domain名字用来做开关，如果一台机器上部署了多个应用，可以指定把一个应用的监控关闭。

```xml
<config mode="client">
          <servers>
             <server ip="10.66.13.115" port="2280" />
         </servers>
</config>
```

alpha、beta这个配置需要自己在此目录添加
预发以及生产环境这个配置需要通知到对应OP团队，让他们统一添加，自己上线时候做下检查即可
a、10.66.13.115:2280端口是指向测试环境的cat地址
b、配置可以加入CAT的开关，用于关闭CAT消息发送,将enabled改为false，如下表示将mobile-api这个项目关闭

```xml
<config mode="client">
          <servers>
             <server ip="10.66.13.115" port="2280" />
         </servers>
         <domain id="mobile-api" enabled="false"/>
 </config>
```

#### 配置

1\) macula.properties

```
#监控开启，默认是true，不开启监控
monitor.disabled = false
```

2\) web.xml，将下面的Filter加在最前面的filter中

```xml
    <!-- Cat Filter -->
    <filter>
        <filter-name>maculaPluginsCat</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <!-- Cat Filter Mapping -->
    <filter-mapping>
        <filter-name>maculaPluginsCat</filter-name>
        <servlet-name>appServlet</servlet-name>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>
```

```
这将开启对所有URL请求的监控，但是默认排除了资源文件。
```

3\) log4j.properties

```
### cat appender ###
log4j.appender.cat=org.macula.plugins.cat.log4j.CatAppender

### set log levels - for more verbose logging change 'info' to 'debug' ###
log4j.rootLogger=WARN, stdout, fileout, cat
```

开启log4j发送到Cat，只有Error或以上级别的日志会发送

4\) Druid DataSource配置

```xml
        <property name="proxyFilters">
            <list>
                <bean class="org.macula.plugins.cat.druid.CatFilter" />
            </list>
        </property>
```

druid数据源中添加上述配置开启对SQL的监控。

5\) Spring Service监控
依赖macula-plugins-cat插件默认会开启对@Service注解的方法的监控

6\) Dubbo监控
给dubbo配置上CatConsumerFilter和CatProviderFilter即可完成对dubbo分布式访问的监控，默认已经开启了这两个Filter。

同时，在消费端需要配置：

```xml
<dubbo:reference id="registryService" interface="com.alibaba.dubbo.registry.RegistryService" check="false" />
```

如果是直接连接没有注册中心的，不能配置上述registryService，通过给reference添加provider参数来配置。

```xml
<dubbo:reference id="demoService" interface="org.macula.plugins.dubbo.test.api.DemoService" >
    <dubbo:parameter key="provider" value="hello-app"/>
</dubbo:reference>
```

上述配置的主要目的就是让消费方知道调用的Service的Application Name是什么，这样CAT监控的时候就可以相互关联起来。

7\) 埋点监控
请查看Cat文档，特别是业务指标监控，需要在Cat后台添加相应的指标名称，然后才能够显示。

8\) 转义送到CAT的URL

在你的webapp中找到xxx-servlet.xml,添加如下配置

```
<bean class="org.macula.plugins.cat.web.RequestToNameBridge" /> 
```

然后在xxx-app.xml中添加如下配置

```
<bean class="org.macula.plugins.cat.web.DefaultRequestToNameImpl" /> 
```

当然，你也可以自己实现真实的URL请求地址的转义，具体可以参考DefaultRequestToNameImpl的实现方式。

#### 从macula-plugins-monitor迁移到CAT监控

如果之前有使用macula-plugins-monitor监控，需要现在web.xml中注释掉如下配置

```xml
  <!-- Monitor HTTP -->
    <!-- 
    <filter>
        <filter-name>maculaPluginsMonitoring</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    -->
     <!--
    <filter-mapping>
        <filter-name>maculaPluginsMonitoring</filter-name>
        <servlet-name>appServlet</servlet-name>
    </filter-mapping>
    -->
    <!-- 
    <listener>
        <listener-class>org.macula.plugins.monitor.SessionListener</listener-class>
    </listener>
     -->    
```

在你的webapp的pom.xml中，取消对macula-plugins-monitor的依赖

```xml
    <!-- 
    <dependency>
        <groupId>org.macula.plugins</groupId>
        <artifactId>macula-plugins-monitor</artifactId>
    </dependency>
    -->
```

搜寻DefaultRequestToNameImpl和RequestToNameBridge，将这两个bean定义拿掉。

### 移动应用监控

TODO

### 浏览器监控

TODO

