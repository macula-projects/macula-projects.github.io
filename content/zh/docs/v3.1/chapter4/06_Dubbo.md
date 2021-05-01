---
type: docs
title: "Dubbo"
linkTitle: "Dubbo"
weight: 6
description: "本文介绍Dubbo使用"
---

如果你要编写dubbo服务，建议如下分包：

###macula-cart-api
对外的服务接口，需要在src/main/resources/META-INF/macula-cart-api-consumer.xml中，引入

```
<dubbo:reference id="demoApi" interface="org.macula.cart.api.DemoApi" version="2.0.0" check="false" />
```
这样第三方引用这个API包的时候，加载xml就可以自动获取到dubbo的reference

###macula-cart-api-impl
接口实现，这里可以调用repository做数据库的操作。定义如下配置文件：
* src/main/resources/META-INF/macula-cart-api-impl-app.xml
```
<context:component-scan base-package="org.macula.cart.api.impl" />
```
用于扫描注解。

* src/main/resources/META-INF/macula-cart-api-impl-provider.xml
```
 <dubbo:service interface="org.macula.cart.api.DemoApi" ref="demoApiImpl" version="2.0.0" />
```
用于定义dubbo服务

* src/main/resources/configs/applicationContext-app.xml

    *与webapp中的作用相同*
* src/main/resources/包含
    1. applicationContext-root.xml
    2. macula.properties
    3. log4j.properties
    4. ehcache.xml

作用基本与webapp中的相同，只是需要注意数据源的配置

###macula-cart-repository
数据存取层

###macula-cart-result
数据返回结果VO

###*注意*
建议dubbo服务不要依赖macula-base包，这样的话，就不需要定义macula数据源到dubbo中，也就是说dubbo服务可以不用访问macula数据库。

