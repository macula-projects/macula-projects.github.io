---
type: docs
title: "数据存取层"
linkTitle: "数据存取层"
weight: 4
description: "本文介绍基于Spring Data JPA的数据存取层"
---

在数据存取层，与传统的DAO层的实现不同，这里引入[spring-data-jpa](http://docs.spring.io/spring-data/jpa/docs/current/reference/html/)开源框架，可实现部分接口只定义接口，而不用编写实现，可减少编码的工作量。

### Spring Data JPA数据存取接口

Spring Data JPA数据存取接口JpaRepository默认可实现下列功能：

例 6.1. JpaRepository 接口

```java
public interface JpaRepository<T, ID extends Serializable> {

    T save(T entity); 

    T findById(ID primaryKey); 

    List<T> findAll(); 

    Page<T> findAll(Pageable pageable); 

    Long count(); 

    void delete(T entity); //  

    boolean exists(ID primaryKey); //  

    // & more functionality omitted.

}
```

在现有的macula框架下，因为使用Spring-Data的JPA模块来构建Repository，所以，对于一般的存取层接口来说，直接通过继承该接口的方式来实现通用存取接口的实现。

例 6.2. 比如实现Application领域模型的存取接口定义为：

```java
public interface ApplicationRepository extends JpaRepository<JpaApplication, Long> {


    public JpaApplication findByAppId(String appId);

}
```

这里ApplicationRepository通过继承JpaRepository，并通过指定泛型`<JpaApplication,Long>来标识JpaRepository的操作对象，即完成了Application领域模型的基本存取接口定义。`

对于增加的findByAppId接口定义，将在下一节介绍。

### MaculaJpaRepository接口

为了能在Spring-Data的基础上具有一定的扩展性，Macula平台基于JpaRepository定义了MaculaJpaRepository接口，并增加了getEntityManager等方法，用来提高JpaRepository的可操作性。

_**重要**_

_为了适应Macula平台的扩展性，在编写Repository时，需要继承MaculaJpaRepository，而不是JpaRepository。_

### Spring自动扫描

通过Spring-Data的自定义命名空间，可将上述的JpaRepository定义的接口直接转化为spring bean，而不需要编写实际的实现类。

例 6.3. Macula平台下定制的Repository -Factory实例：

```xml
<jpa:repositories base-package="org.macula.base.**.repository" entity-manager-factory-ref="entityManagerFacotry"
        transaction-manager-ref="transactionManager" factory-class="org.macula.core.repository.MaculaJpaRepositoryFactoryBean" />
```

_**重要**_

_请注意这里的配置与Spring-Data中介绍的一样，但factory-class请使用macula平台编写的FactoryBean，它主要完成了在自定义接口与实现时，如果使用了@Transactional或EntityManager对象，将会使用配置中的transaction-manager-ref与entity-manager-factory-ref配置的Bean作为注入，这样可保证自定义接口与原接口使用相同的jpa entityManager与事务处理。_

对于这里定义的repository命名中，各属性值的说明如下：

* base-package：指明扫描时的目录，可以允许通过\*\*的方式，定义匹配的目录。这里请在实际使用中，使包的扫描范围尽量精确，以加快扫描进度以及减少不必要的Spring Bean扫描。

* entity-manager-factory-ref：这里指明JpaRepository以及自定义接口中所使用的JPA EntityManagerFactory Bean的名字，通过这里的定义，可实现在多个JPA EntityManagerFactory Bean定义的情况下，引入正确的Bean实例。

* transaction-manager-ref：该属性指明在JpaRepository与自定义接口中，使用到了@Trasactional注解时，所使用的事务。在JpaRepository中，已经存在了定义的@Transactional注解的接口，所以为了避免在定义了多个TransactionManager的情况下，能正确引入响应的事务处理Bean，可通过该属性来定义。

* factory-class：可以看到，这里我们只定义了需要的接口，而不需要编写实现，而通过接口转化为Spring可识别的Bean，采用了Spring的FactoryBean（Bean工厂）的模式，所以需要定义一个用来生成Bean实例的工厂Class，这里，已经由Macula框架完成了该Bean工厂的实现，即org.macula.core.repository.MaculaJpaRepositoryFactoryBean，该Bean扩展自Spring-Data对应的Bean工厂，如有兴趣可继续查看Spring-Data的实现。

_**重要**_

_这里只定义了Repository的接口，即可通过Spring-Data的一个扫描即可生成对应的Bean的实例，看似非常神奇，实际上使用了Spring的FactoryBean的构建方式，通过工厂来返回了一个JpaRepository的实现来作为我们定义的接口的实现，而自定义的接口，则通过命名上查找对应的Class Implement来构建custom的实现。  
这里repositories标签扫描的规则是：_

* _接口扩展了JpaRepository，即extends JpaRepository。_
* _接口如果通过注解@NoRepositoryBean，则标识不用扫描该接口_

### 接口方法

除开已有的JpaRepository中已有的接口定义不需要再编写实现类外，对于查询部分接口，也同样不需要编写实现，但需要查询方法定义名称定义符合一定的规范。

**例 6.4. 根据findBy后面的属性名查询：**

```java
public List<Person> findByLastname(String lastname);
```

该方法标识采用Lastname属性查询Person列表，lastname的属性值为参数。

**例 6.5. 根据findBy后的属性名分页、排序查询：**

```java
public Page<User> findByLastname(String lastname, Pageable pageable);
public List<User> findByLastname(String lastname, Sort sort);
```

**例 6.6. 根据findBy后的多个属性查询：**

```java
public List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
```

**例 6.7. 根据findBy后的属性的子属性查询：**

```java
public List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

该方法通过address.zipCode来查询Persion对象列表

_**关于扩展JpaRepository接口中可定义的方法而不用编写实现代码的部分，可查看Spring-Data中JPA部分（data-jpa）的文档。**_

### 自定义接口与实现

对于一些业务需求在以上介绍的在接口定义即可完成的，不需要编写自定义接口，否则需要编写自定义的接口并实现自定义接口。

**例 6.8. 自定义接口：UserRepositoryCustom**

```java
public interface UserRepositoryCustom {


    public void someCustomMethod(User user);

}
```

**例 6.9. 自定义接口实现**

```java
public class UserRepositoryImpl implements UserRepositoryCustom {

  public void someCustomMethod(User user) {
    // Your custom implementation
  }

}
```

**例 6.10. 对外使用的接口：UserRepository**

```java
public interface UserRepository extends MaculaJpaRepository<User, Long>, UserRepositoryCustom {

  // Declare query methods here

}
```

参考这个流程，可以看出，只有针对特殊需要的接口，才需要编写额外的接口。

针对Java的特殊性，实现类必须实现完整的接口定义，所以对于自定义方法的部分，需要将自定义方法独立定义成一个接口类，然后将最终需要使用的接口继承该接口即可。

对于接口的实现类名，有一定的规则，默认情况下，使用接口类名+Impl的方式命名实现类，才可以通过定义自动检测到，在macula平台开发下，强制要求按这个命名规则命名。

### 自定义接口中的EntityManager和TransactionManager

为了保证repositories命名空间定义的spring自动扫描能准确的将EntityManager和TransactionManager注入到自定义的实现中，对自定义实现类需要做下列规范：

* 自定义实现类不能标记@Service、@Repository、@Component等注解

* 自定义实现类可通过@Autowire在注入需要的bean实例

* 自定义实现需要使用EntityManager时，不可通过@PersistentContext注入entityManager，只能通过实现JpaEntityManagerAware接口中的setEntityManager来获取entityManager的注入。

  其中JpaEntityManagerAware的接口标记如下为：

  ```java
  public interface JpaEntityManagerAware {

        public void setEntityManager(EntityManager entityManager);

    }
  ```

  _**注意**_

* 该接口由macula平台提供，并由repositories中定义的factory-class：org.macula.core.repository.MaculaJpaRepositoryFactoryBean来正确处理，为了保证自定义实现能灵活的替换EntityManager而做出的扩展。\*

* 自定义实现中的@Transactional，可直接定义在接口中，但在@Transactional的定义中，不要指定transactional使用的TrasactionManager的名称，道理和使用EntityManager相同，都由Macula平台的factory-class来统一处理。

对于Repository层的开发，这里主要介绍了macula平台在Spring-Data下做出的扩展，更多的示例可参考macula平台提供的插件模块和示例模块，对于Spring-Data自身提供的功能，可以查看Spring-Data的官方文档。

### 使用TemplateQuery注解

Macula扩展了spring-data-jpa的功能，除了原先可以支持的@Query、@NamedQuery等方法上的注解，Macula提供了TemplateQuery注解。  
原先的注解SQL语句不支持动态条件，不能写if等表达式。TemplateQuery注解支持在注解中或者模板文件中编写SQL语句，可以使用freemarker语法编写，具体使用方式如下：

```java
package org.macula.core.test.repository;
...
public UserRepository extends MaculaJpaRepository<User> {
    ...
    @TemplateQuery
    public Page<UserVo> findByLastNameVo(@Param("lastName") String lastName, Pageable pageable);

    @TemplateQuery
    public Page<User> findByLastNameMap(@Param("data") Map<String, Object> data, Pageable pageable);

    @TemplateQuery("select * from MY_USER u where 1=1" +
           "<#if (data.lastName)??>" +
          "    and u.last_name = :data.lastName" + 
          "</#if>" +
          "<#if firstNames??>" +
          "    and u.first_name in (:firstNames)" +
          "</#if>")
    public Page<User> findByLastNameMapAndList(@Param("data") Map<String, Object> data, 
                    @Param("firstNames") List<String> firstNames, Pageable pageable);

    @TemplateQuery
    public Page<User> findByLastNameMapAndListx(@Param("data") Map<String, Object> data, 
                    @Param("firstNames") List<String> firstNames, Pageable pageable);

    @TemplateQuery
    public Page<User> findByLastNameMapy(@Param("data") Map<String, Object> data, Pageable pageable);

    @TemplateQuery
    public Page<User> findByLastNameMapAndListy(@Param("data") Map<String, Object> data, 
                    @Param("firstNames") List<String> firstNames, Pageable pageable);
}
```

同时，没有在@TemplateQuery value中写的SQL需要在文件中编写对应的SQL模板，支持在两个位置编写SQL模板：

1）src/resources/sqls/module-name/{domainName}.xml中编写SQL，文件命名是Domain类的全名称加上.xml（3.1有可能废弃）

2）src/java/{repositoryPackage}/{repositoryName}.xml中编写SQL，文件放在该TemplateQuery方法所属的Repository类路径下，和Repository类名称一致，以xml结尾。例如：src/java/org/macula/core/test/repository/UserRepository.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<sqls xmlns="http://www.maculaframework.org/schema/repository"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.maculaframework.org/schema/repository http://macula.top/schema/repository/macula-repository-1.0.xsd">

    <sql name="findByLastNameVo">
        <![CDATA[
          select u.first_name, u.last_name from MY_USER u where u.last_name = :lastName
        ]]>
    </sql>

    <sql name="findByLastNameMap">
        <![CDATA[
          select * from MY_USER u where u.last_name = :data.lastName
        ]]>
    </sql>

    <sql name="findByLastNameMapAndListx">
        <![CDATA[
          select * from MY_USER u where 1=1
           <#if (data.lastName)??>
              and u.last_name = :data.lastName 
          </#if>
          <#if firstNames??>
              and u.first_name in (:firstNames)
          </#if>
        ]]>
    </sql>
</sqls>
```

_**注意**_

* findByLastNameVo演示了通过Vo返回数据；
* findByLastNameMap演示了通过Map传递参数给SQL语句，通过DomainClass返回数据；

* TemplateQuery的查询结果会自动转换到你要返回的类型，但是返回类型中的属性名称与数据库列名称必须对应起来，默认会将返回类型的属性名称的大写字母转换为\_加小写，比如firstName会转换为first\_name与数据库列对应，数据库的列也会统一转换为小写；

* 方法中的参数都需要@Param标识参数名称，以便和SQL语句中的参数占位符对应，如果参数类型是Map或者Bean，则SQL语句中的参数名称需要是 参数名称.属性名称，比如data.lastName；

* FreeMarker的语法全部可以用在SQL语句中，可以解析的参数都是来源于方法中的参数值，所有参数值都会放到一个Map中传递给FreeMarker，同样，Bean或者Map参数需要加上他们的名字，比如data.lastName

#### sftl格式模板

为了减少编写XML模板压力，从3.0.1开始，除了支持XML格式模板外，还支持sftl模板，格式如下，具体放置路径同XML模板，只是把后缀改为.sftl：

```
--findByLastNameMapy
    select * from MY_USER u where u.last_name = :data.lastName

--findByLastNameMapAndListy
    select * from MY_USER u where 1=1
    <#if (data.lastName)??>
      and u.last_name = :data.lastName 
    </#if>
    <#if firstNames??>
      and u.first_name in (:firstNames)
    </#if>
```

#### TemplateQuery使用优先级

TemplateQuery支持通过注解、文件、配置属性提供SQL，如果出现RepositoryName加MethodName重复，则配置属性优先，注解次之，文件中的SQL最后。

#### 使用通用配置热修复TemplateQuery的SQL

线上运行的系统有时发现TemplateQuery的SQL写得有问题，可以通过Macula支持的基于zookeeper的通用配置临时添加一个属性热修复。具体属性KEY是macula.templateQuery.{repositoryName}.{methodName}，内容是SQL模板。请谨慎使用，待程序修复后要即时删除该配置，否则永远是这个配置优先。

