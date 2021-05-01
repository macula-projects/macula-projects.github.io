---
type: docs
title: "配置文件"
linkTitle: "配置文件"
weight: 1
description: "本文介绍了Macula的配置文件"
---

基于Macula开发的项目，您需要关注的配置文件位于您的webapp工程下，包括：

* **web.xml配置**

  * 一般情况下参考框架的配置，或者你的项目不添加这个文件，直接使用框架的。

* **Spring配置**

  * applicationContext-root.xml  数据库相关、Redis相关等需要连接外部资源的配置
  * configs/applicationContext-app.xml 事务、JPA等相关配置，与环境无关
  * configs/servletContext-app.xml MVC层面自定义配置
  * src/main/resources/META/spring/macula-\*-app.xml 非MVC层的各自模块配置文件
  * src/main/resources/META/spring/macula-\*-servlet.xml MVC层的各自模块配置文件 

* **属性配置文件**

  * **macula.properties**  Macula框架配置
  * **freemarker.properties** FreeMarker配置
  * **log4j.properties**  Log4j配置
  * druid-macula.properties Druid数据源相关的配置
  * druid-xxx.properties
  * app.properties 该文件用于独立的模块中，用来覆盖macula.properties中的值，比如front和mobile分开打包，并且appName可能不一样，则分别在front和mobile包中引入app.properties。

* ehcache.xml

  * ehcache配置

## web.xml配置

### 1\) J2EE项目下，web.xml中的Spring通过Listener载入

```
<listener>
        <listener-class>org.macula.core.listener.MaculaContextLoaderListener</listener-class>
</listener>
```

Listener需要设置的参数

```
<context-param>
        <param-name>locatorFactorySelector</param-name>
        <param-value>classpath:/configs/applicationContext-ref.xml</param-value>
    </context-param>
    <context-param>
        <param-name>parentContextKey</param-name>
        <param-value>MaculaContextRoot</param-value>
    </context-param>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:/configs/applicationContext-app.xml,classpath:/configs/applicationContext-macula.xml,classpath:/configs/applicationContext-security.xml</param-value>
    </context-param>
```

### 2\) Spring MVC包括web.xml中对Spring Filter的定义以及对应的Spring配置信息定义。

在web.xml中定义：

```
<servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.macula.core.mvc.MaculaDispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:/configs/servletContext-mvc.xml, classpath:/configs/servletContext-app.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

_**重要**_

_应用系统开发中，通过web.xml设置的Spring加载的参数值，必须按照上面的代码执行，即：文件名、目录名必须按指定的代码定义。_

_web.xml的其他配置请参考macula-plugins-webapp.war中的web.xml_

## Spring配置

Macula开发平台基于Spring框架开发，使用者需要了解Spring的基本原理以及使用方法（参见附录Spring Framework），本章介绍在Macula开发平台中，所需要配置/修改的Spring相关配置信息。

### 1\) applicationContext-root.xml

该文件放置路径与applicationContext-ref.xml中配置的classpath:applicationContext-root.xml一致，即必须放在src/main/resources目录。

应用系统所使用的数据库设置必须在此文件中定义。下面是参考的代码信息：

```
<beans>    
       <context:annotation-config />
       <context:component-scan base-package="org.macula.core.configuration" />
       <import resource="classpath*:/META-INF/spring/macula-*-root.xml" />

       <!-- 数据源的配置 -->
       <bean id="macula_dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
        ...
       </bean>    
       <bean id="macula-cart_dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
       ...
       </bean>        
       <!-- REDIS配置 -->
       <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
           <property name="connectionFactory" ref="redisConnectionFactory" />
       </bean>
       <alias name="redisTemplate" alias="cacheRedisTemplate"/>
       <alias name="redisTemplate" alias="transportRedisTemplate"/>
   </beans>
```

* 上述配置文件首先配置了框架的Configuration扫描，这里不需要修改，同时，如果需要放在根环境预先加载的spring配置可以放在/src/main/resources/macula-\*-root.xml文件中。
* 定义了两个数据源，一个指向框架，一个指向业务，具体可以根据需要修改
* 配置了redis等其他和环境相关的配置

### 2\) configs/applicationContext-app.xml

该文件设置应用所需要包含的其他Spring配置文件，以及对系统所涉及到的公共信息Bean的定义，如：Jpa定义、Transaction定义等，该文件严禁定义更为复杂的模块信息的Bean，应有import方式导入。  
   对于引入的子模块的Spring信息，必须如下定义：

```
<beans>
       <import resource="classpath*:/META-INF/spring/macula-*-app.xml" />
       <context:component-scan base-package="org.macula.core.config,org.macula.core.config,org.macula.cart.**.config">
           <context:include-filter type="annotation" expression="org.springframework.context.annotation.Configuration"/>
           <context:include-filter type="assignable" expression="org.macula.core.config.MaculaAppConfig"/>
       </context:component-scan>

       <bean id="abstractEntityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
           abstract="true">
           <property name="jpaVendorAdapter">
               <bean class="org.macula.core.hibernate.HibernateJpaVendorAdapter">
                   <property name="database" value="#{T(org.macula.Configuration).getDatabase()}" />
                   <property name="showSql" value="#{T(org.macula.Configuration).getShowSql()}" />
                   <property name="generateDdl" value="#{T(org.macula.Configuration).getGenerateDdl()}" />
               </bean>
           </property>
           <property name="jpaProperties">
               <props>
                   <prop key="hibernate.ejb.event.post-update">org.macula.core.hibernate.audit.AuditedEventListener</prop>
                   <prop key="hibernate.ejb.event.post-delete">org.macula.core.hibernate.audit.AuditedEventListener</prop>
               </props>
           </property>
       </bean>

       <!-- Macual Schema -->
       <!-- Macula Entity Manager -->
       <bean id="entityManagerFactory_macula" parent="abstractEntityManagerFactory">
           <property name="persistenceUnitManager">
               <bean class="org.springframework.orm.jpa.persistenceunit.DefaultPersistenceUnitManager">
                   <property name="defaultPersistenceUnitName">
                       <value>macula</value>
                   </property>
                   <property name="defaultDataSource" ref="macula_dataSource" />
                   <property name="packagesToScan">
                       <array>
                           <value>org.macula.base.app.domain</value>
                           <value>org.macula.base.data.domain</value>
                           <value>org.macula.base.acl.domain</value>
                           <value>org.macula.plugins.rule.domain</value>
                       </array>
                   </property>
               </bean>
           </property>
       </bean>

       <bean id="transactionManager_macula" class="org.springframework.orm.jpa.JpaTransactionManager">
           <property name="entityManagerFactory" ref="#{T(org.macula.Configuration).getEntityManagerFactoryName()}" />
       </bean>

       <!-- @Transaction -->
       <tx:advice id="maculaTxAdvise" transaction-manager="transactionManager_macula" />
       <aop:config>
           <aop:pointcut id="maculaPointcut"
               expression="execution(* org.macula..*.*(..)) and !execution(* org.macula.samples..*.*(..)) and @within(org.springframework.stereotype.Service)" />
           <aop:advisor advice-ref="maculaTxAdvise" pointcut-ref="maculaPointcut" />
           <aop:aspect id="exceptionAspect" ref="exceptionHandler">
               <aop:after-throwing pointcut-ref="maculaPointcut" method="doAfterThrowing" throwing="ex" />
           </aop:aspect>
       </aop:config>


       <bean id="jdbcTemplate_macula" class="org.springframework.jdbc.core.JdbcTemplate">
           <constructor-arg index="0" ref="macula_dataSource" />
       </bean>

       <!-- macula-cart Schema -->
       <!-- macula-cart Entity Manager -->
       <bean id="entityManagerFactory_macula-cart" parent="abstractEntityManagerFactory">
           <property name="persistenceUnitManager">
               <bean class="org.springframework.orm.jpa.persistenceunit.DefaultPersistenceUnitManager">
                   <property name="defaultPersistenceUnitName">
                       <value>macula-cart</value>
                   </property>
                   <property name="defaultDataSource" ref="macula-cart_dataSource" />
                   <property name="packagesToScan">
                       <array>
                           <value>org.macula.cart.domain</value>
                       </array>
                   </property>
               </bean>
           </property>
       </bean>

       <bean id="transactionManager_macula-cart" class="org.springframework.orm.jpa.JpaTransactionManager">
           <property name="entityManagerFactory" ref="entityManagerFactory_macula-cart" />
       </bean>    

       <!-- @Transaction -->
       <tx:advice id="macula-cartTxAdvise" transaction-manager="transactionManager_macula-cart" />
       <aop:config>
           <aop:pointcut id="macula-cartPointcut"
               expression="execution(* org.macula.cart..*.*(..)) and @within(org.springframework.stereotype.Service)" />
           <aop:advisor advice-ref="macula-cartTxAdvise" pointcut-ref="macula-cartPointcut" />
           <aop:aspect id="exceptionAspect" ref="exceptionHandler">
               <aop:after-throwing pointcut-ref="macula-cartPointcut" method="doAfterThrowing" throwing="ex" />
           </aop:aspect>
       </aop:config>

       <bean id="jdbcTemplate_macula-cart" class="org.springframework.jdbc.core.JdbcTemplate">
           <constructor-arg index="0" ref="macula-cart_dataSource" />
       </bean>

       <!-- i18n resources -->
       <bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
           <property name="basenames">
               <list>
                         ...
               </list>
           </property>
           <property name="defaultEncoding" value="utf-8" />
           <property name="fallbackToSystemLocale" value="false" />v
       </bean>

       <aop:aspectj-autoproxy />
</beans>
```

* 对于子模块的Spring信息，必须放置在src/main/resources/META-INF/spring目录下，并严格按照macula-\*-app.xml命名配置文件。
* 如果需要子模块支持@Configuration配置，注意要修改上述第三行，扫描放配置类的包，只修改org.macula.cart.\*\*.config；
* 原则上只需要修改上述示例中的macula-cart相关的配置部分，macula框架相关部分禁止修改，当然如果框架的表和业务的表在一个库，上述配置可以合并。
* 另外，国际化的资源文件需要记得添加在mesageSource这个bean中。

### 3\) configs/servletContext-app.xml

```
<beans>
    <import resource="classpath*:/META-INF/spring/macula-*-servlet.xml" />
    <context:component-scan base-package="org.macula.core.config, org.macula.base.config, org.macula.cart.**.config">
        <context:include-filter type="annotation" expression="org.springframework.context.annotation.Configuration"/>
        <context:include-filter type="assignable" expression="org.macula.core.config.MaculaServletConfig"/>
    </context:component-scan>

    <!-- 这里需要根据系统是admin、front、mobile作出修改 -->
    <!-- 
    <mvc:view-controller path="/" view-name="redirect:/admin" />
    -->
</beans>
```

* 子模块MVC层面的配置全部放在/src/main/resources/META-INF/spring/macula-\*-servlet.xml中
* 如果需要子模块支持@Configuration配置，注意要修改上述第三行，扫描放配置类的包，只修改org.macula.cart.\*\*.config；

### 4\) 各模块配置文件

按照前面的叙述，您可以在src/main/resources/META-INF/spring/macula-_-app.xml或则macula-_-servelt.xml中配置Spring。

* app部分主要配置domain、respository、service层；
* servlet主要配置controller层，MVC的东西；

您还可以通过@Configuration注解配置

* app部分的配置类需要继承MaculaAppConfig类；
* servlet部分的配置类需要继承MaculaServletConfig类。    

## 属性配置文件

### 1\) macula.properties

```
#应用所属分组(根据需要修改，同一个appGroup的应用会共享会话，广播事件也可以相互传递)
macula.appGroup = macula-cart
#应用名称
macula.appName = macula-cart
#应用终端类型(用户在这里设置类型，这样可以在同一个AppGroup中实现不同类型的登录互不影响，否则同一个appGroup登录时会踢出另一个同名用户，从而无法实现手机端登录不影响PC）
#macula.terminalType = PC,MT(移动终端),MOBILE(手机)

#cas统一认证配置项
macula.casServerService = https://testuim.infinitus.com.cn
macula.casClientService = http://localhost:8080/macula-cart-webapp

#静态文件在浏览器的缓存时间，单位毫秒，生产环境可以设置为1年
macula.resourceCachePeriod = 60000

#静态文件服务器地址，默认是本应用下的目录，适用CDN加速，分离静态文件
#macula.resourceHost = http://img-cdn.infinitus.com.cn

#cache manager的名字，默认使用系统定义的cacheManger，用户可以自己定义cacheManager，在这里指定名称
#macula.cacheManagerName = cacheManager

#macula框架的entityManagerFactory，默认使用系统定义的entityManagerFactory_macula，用户可以自己定义entityManagerFactory，在这里指定名称
#macula.entityManagerFactoryName = entityManagerFactory_macula

#macula框架的transactionManager，默认使用系统定义的transactionManager_macula，用户可以自己定义transactionManager，在这里指定名称
#macula.transactionManagerName = transactionManager_macula

#功能是否作为角色，用在需要对单个功能授权的地方
macula.actionAsRole = false

#每个用户的最大会话数
macula.maximumSessions = 1

#验证码出现的条件
macula.captchaFailedTimes = 3

#会话过期时间，单位秒
macula.sessionTime = 600

#系统运行模式 dev/test/prd
macula.runMode = dev

#是否记录登录日志，默认是false
macula.loginLog = true

#是否记录访问日志，默认是false
macula.accessLog = true

#访问日志记录队列深度，默认是2000
#macula.logQueueCapacity = 2000

#是否关闭事件广播
#macula.disableBroadcast = true

#事件广播方式，默认是redis，可以配置http、redis、zookeeper
macula.events.transport = redis

#配置需要保护的地址
macula.securityUrlPattern = /.*
#macula.securityUrlRole = ROLE_SECURITY

#可以匿名访问的地址（白名单）
macula.publicUrlPattern = /|/index.*|/login.*|/logout.*|/resources.*|/views.*|/.*public.*|/error/.*|/.*/blank.*|/.*/ajaxforward.*

#设置网站流量统计功能开关，页面根据这个设置决定是否加载统计代码
#macula.statsMode = baidu,google,none

#设置系统使用的界面库，默认是macula 1.0提供的界面库。如果用逗号隔开，则freemarker会按照顺序加载指定ui后缀的ftl文件
#比如如下设置，则freemarker会加载xxx_mower.ftl，如果找不到xxx_mower.ftl则加载xxx.ftl
macula.uiList = mower

#日期时间格式
pattern.datetime = yyyy-MM-dd HH:mm:ss
pattern.date = yyyy-MM-dd
pattern.time = HH:mm:ss
pattern.number = #

jpa.showSql = true
jpa.generateDdl = false

#######环境设置########################
jpa.database = MYSQL
#macula.disableBroadcast = true
#####################################

#初始菜单起始设置
macula.adminRootMenu = ADMIN_GROUP
macula.frontRootMenu = FRONT_GROUP
#macula.mobileRootMenu = MOBILE_GROUP

#在启动命令中添加该配置，可以作为macula.properties中加密串的密钥
#java xxx -Dmacula.secretKey = xxxx
#在启动命令添加该配置，设置macula.appInstance
#java xxx -Dmacula.appInstance = xxxx
#在启动命令添加该配置，可以设置启动时加载的环境文件
#java xxx -Dmacula.profile = xxxx
```

_**重要**_

`上述所有配置都可以通过启动命令行设置来覆盖上述默认配置。`

### 2\) log4j.properties

没有什么特殊的，需要提醒的是生产环境不要设置为DEBUG级别，防止日志文件太大。

### 3\) freemarker.properties

```
default_encoding=UTF-8
#template_exception_handler=ignore
#template_exception_handler=debug
#template_exception_handler=html_debug
#template_exception_handler=rethrow

#开发环境可以注释掉下面两行，这样修改ftl文件可以立即生效，不用重启tomcat
cache_storage=strong:5000, soft
template_update_delay=86400

auto_import="/spring.ftl" as spring, "macula.ftl" as macula, "/layout.ftl" as layout, "/ui.ftl" as ui
```

### 4\) druid-macula.properties

druid数据源的配置文件，配合applicationContext-root.xml中dataSource,不同环境应该有不同的配置

```
# 基本属性 url、user、password
url=jdbc:mysql://127.0.0.1:3306/macula3?useUnicode=true&amp;characterEncoding=utf-8&amp;zeroDateTimeBehavior=convertToNull
username=root
password=mysql

# 配置初始化大小、最小、最大
initialSize=10
minIdle=10
maxActive=100

# 配置获取连接等待超时的时间
maxWait=60000

# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
timeBetweenEvictionRunsMillis=60000

# 配置一个连接在池中最小生存的时间，单位是毫秒
minEvictableIdleTimeMillis=300000
validationQuery=SELECT 1 from dual
testWhileIdle=true
testOnBorrow=false
testOnReturn=false

# 打开PSCache，并且指定每个连接上PSCache的大小
poolPreparedStatements=true
maxPoolPreparedStatementPerConnectionSize=20

# 不加密密码
config.decrypt=false
```

## EHCACHE配置

```
<defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true"
        />

    <cache name="instanceCache" 
        maxElementsInMemory="3000"
        eternal="false"
        timeToIdleSeconds="1800"
        timeToLiveSeconds="3600"
        overflowToDisk="false"
        statistics="true"
        />
```

上述instanceCache是给Spring Cache使用的，注入到cacheManager中的，一般情况下不要再添加任何配置，可能需要根据应用的需要修改缓存大小、时间等。

## 多环境配置问题

一般应用程序在开发、测试、生产的配置都是不一样的，框架支持在启动时添加参数来选择不同的环境参数，具体如下：

* 修改web.xml中的配置，将ContextLoaderListener改为MaculaContextLoaderListener，原来的MaculaConextListener删除，最终变成如下：

  ```xml
  <listener>
        <listener-class>org.macula.core.listener.MaculaContextLoaderListener</listener-class>
    </listener>

    <listener>
        <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class>
    </listener>

    <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>
  ```

* 启动时在命令行添加-Dmacula.profile=xxx，其中xxx表示环境路径

* 在您的webapp或者api-impl包的configs目录下根据xxx建立相应的目录，系统会自动从该目录中加载

  * macula.properties
  * freemarker.properties\(不依赖macula-base的应用不加载这个文件\)
  * log4j.properties

* 在applicationContext-root.xml中，可以通过Spring的Profile来区分环境配置，如：

  ```xml
  <beans profile="local"> 
      <bean id="redisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
          <property name="hostName" value="localhost" />
      </bean>
  </beans>
  ```

  则上段配置只有当环境是local时才会加载，profile可以写入多个环境，用逗号隔开，如果profile中有default，则没有配置环境属性时也会加载，如

  ```xml
  <beans profile="default,dev"> 
        <bean id="redisConfig" class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
             <constructor-arg index="0" value="mymaster" />
             <constructor-arg index="1">
                 <set>
                     <value>soa-dev01.infinitus.com.cn:26379</value>
                     <value>soa-dev01.infinitus.com.cn:26479</value>
                 </set>
             </constructor-arg>
         </bean>
    </beans>
  ```

* 数据源同样也是配置在applicationContext-root.xml中

  ```xml
  <bean id="macula_dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
         <!-- 配置监控统计拦截的filters -->
         <property name="filters" value="stat,config" />
         <!-- 配置CAT拦截 -->
         <property name="proxyFilters">
             <list>
                 <bean class="org.macula.plugins.cat.druid.CatFilter" />
             </list>
         </property>
         <!-- 配置数据源连接 -->
         <property name="connectionProperties" value="config.file=classpath:#{T(org.macula.Configuration).getProfilePath()}druid-macula.properties" />
   </bean>
  ```

* 其他如MongoDB等配置采用类似方式即可。如果启动时没有加入-Dmacula.profile，则系统会在classpath的根路径下寻找上述properties文件，同时，Configuration.getProfile\(\)和Configuration.getProfilePath\(\)返回空串。

* 在应用的启动脚本 中添加 -Dmacula.projectId参数（例如： -Dmacula.projectId=bupreq-111.yunxiao），框架会在框架中与域名有关的地方，将原本的域名进行转换。如：https://po-dev.infinitus.com.cn -> http://po-bupreq-111.yunxiao.infinitus.com.cn



