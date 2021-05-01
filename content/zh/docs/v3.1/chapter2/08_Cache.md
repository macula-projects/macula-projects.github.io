---
type: docs
title: "缓存服务"
linkTitle: "缓存服务"
weight: 9
description: "本文介绍了缓存的相关配置和使用方式"
---

Macula开发平台的缓存基于Spring-Cache模块，通过EhCache与Redis两大开源实现，在此基础上提供了缓存服务的支持。为了适应Web开发中的不同生命周期数据的需要，在Spring-Cache的基础上，实现了SESSION、INSTANCE、APPLICATION级别的缓存作用域。

在介绍缓存服务前，需要注意的是：

_**重要**_

_缓存中的数据是不可靠的，即缓存中的数据总是有生命周期的，所以通过缓存获取到的数据，并不总是能得到期望中的值，所以程序要考虑在缓存中没有获取到正确数据的情况下，需要能通过其他方式获取，在有需要的情况下，更新缓存数据。_

## Cache作用域

**Cache作用域说明**

| 作用域 | 说明 | 获取方式 |
| :--- | :--- | :--- |
| CacheScope.SESSION | 基于Web容器Session的作用范围，在Session失效后，所缓存的数据将失效 | CacheUtils.getSessionCache\(\)，或通过注入CacheManager cacheManager然后通过cacheManager.getCache\(CacheScope.SESSION\)获取 |
| CacheScope.INSTANCE | 实例级作用范围，在以JVM为周期的缓存。数据缓存有效期的时间通过EhCache配置文件设定 | CacheUtils.getInstanceCache\(\)，或通过注入CacheManager cacheManager然后通过cacheManager.getCache\(CacheScope.INSTANCE\)获取 |
| CacheScope.APPLICATION | 集群级作用范围，独立于运行中的各实例，当前使用Redis来作为缓存服务器。数据缓存有效期为24小时 | CacheUtils.getApplicationCache\(\)，或通过注入CacheManager cacheManager然后通过cacheManager.getCache\(CacheScope.APPLICATION\)获取 |

## Session级Cache

Session级的Cache依赖于Web容器的HttpSession，由于默认框架采用redis保存HttpSession，所以存入的数据要能够序列化，要保证每个用户尽量少的占用系统资源的要求，尽量减少Session级的Cache数据。

在Spring中配置的Bean如下：

```xml
<!-- Session Cache -->
<bean class="org.macula.core.cache.session.SessionCacheFactoryBean">
      <property name="name" value="sessionCache" />
</bean>
```

## Instance级Cache

Instance级表示的是服务器实例级别的Cache，即以JVM为其作用域，缓存数据通过EhCache实现。

对实例级Cache的配置可通过ehcache.xml来实现。

```xml
<cache name="instanceCache" 
        maxElementsInMemory="300"
        eternal="false"
        timeToIdleSeconds="500"
        timeToLiveSeconds="500"
        overflowToDisk="true"
        />
```

* name

  ehcache的cache名，对应于Instance级的Cache名称为instanceCache；

* maxElementsInMemory

  标识在Cache中可存放的数据条目数；

* timeToIdleSeconds

  数据闲置时间，超过闲置时间将被移除

* timeToLiveSeconds

  数据生存时间，即从放入Cache到数据失效的时间


在Spring中配置的Bean如下：

```xml
<!-- Instance Cache -->
<bean class="org.springframework.cache.ehcache.EhCacheCacheManager">
     <property name="cacheManager">
         <bean class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" />
     </property>
</bean>
```

## Application级Cache

Instance级表示的是独立于服务实例的Cache，用于多应用实例间的数据共享缓存。

对实例级Cache的配置可通过redis的Bean来配置，它包括MemcacheFactoryBean的配置与MemcacheClient的配置。

```xml
<bean id="redisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
      <property name="hostName" value="localhost" />
</bean>
<bean id="cacheRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
      <property name="connectionFactory" ref="redisConnectionFactory" />
</bean>

<!-- Application Cache -->
<bean class="org.macula.core.cache.redis.RedisCacheManager">
    <constructor-arg index="0" ref="cacheRedisTemplate" />
    <property name="cacheName" value="applicationCache" />
    <property name="usePrefix" value="true" />
</bean>
```

## Cache接口

Cache接口直接采用Spring-Cache的接口方式。具体可参考org.springframework.cache.Cache接口。

## Cache的总体配置

```
<bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
        <property name="cacheManagers">
            <list>
                <!-- Instance Cache -->
                <bean class="org.springframework.cache.ehcache.EhCacheCacheManager">
                    <constructor-arg index="0">
                        <bean
                            class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" />
                    </constructor-arg>
                </bean>
                <!-- Session Cache -->
                <bean class="org.springframework.cache.support.SimpleCacheManager">
                    <property name="caches">
                        <set>
                            <!-- Session Cache -->
                            <bean class="org.macula.core.cache.session.SessionCacheFactoryBean">
                                <property name="name" value="sessionCache" />
                            </bean>
                        </set>
                    </property>
                </bean>
                <!-- Application Cache -->
                <bean class="org.macula.core.cache.redis.RedisCacheManager">
                    <constructor-arg index="0" ref="cacheRedisTemplate" />
                    <property name="cacheName" value="applicationCache" />
                    <property name="usePrefix" value="true" />
                </bean>
            </list>
        </property>
</bean>
```

## Cache的其他用途

由于该Cache最终为Spring-Cache实现，所以对于Spring-Cache的其他用途，如通过annotation标识方法的缓存等，请具体参见Spring文档。

_**重要**_

使用@Cacheable注解时记得要指明作用域。

