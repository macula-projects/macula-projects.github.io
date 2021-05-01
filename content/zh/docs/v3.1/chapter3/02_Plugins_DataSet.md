---
type: docs
title: "数据相关"
linkTitle: "数据相关"
weight: 2
description: "本文了数据集相关的概念和使用方式"
---

为了便于数据的获取，使用Macula的各种混搭技术，我们引入了数据集的方式来产生数据集合。

数据集具有以下的特点：

1. 数据源可以基于不同的数据库，但每个数据集只能最多使用一个数据源；
2. 数据集可以配置多个不同的串行处理器；
3. 数据集可以通过编程接口编写，也可以通过XML的方式配置；

## 数据源（DataSource）

数据源表示一种数据来源。当前的数据源支持两种来源方式：

1. 数据库

   通过设定Java JDBC所需要的元素，来获取数据库连接。在JDBC环境下，可以设置两种类型的数据库连接：JNDI配置的容器级的数据源和通过JDBC创建的数据源。

2. LDAP

   连接LDAP库


数据源可以在数据库中定义，或者通过XML配置，xml文件放在src/main/resources/xxxx/\*-datasource.xml中：

```
<datasource id="macula_ds" name="MACULA测试">
        <jndi>false</jndi>
        <dataSourceType>DATABASE</dataSourceType>
        <url>jdbc:oracle:thin:@192.168.0.180:1521:dstest</url>
        <driver>oracle.jdbc.driver.OracleDriver</driver>
        <username>macula</username>
        <password>macula</password>
        <validationQuery>select 1 from dual</validationQuery>
    </datasource>
```

## 枚举\(DataEnum\)

提供统一的枚举数据表，表名为MA\_BASE\_DATA\_ENUM。

## 数据参数\(DataParam\)

可以通过管理界面定义在数据库中或者通过XML配置，xml文件放在src/main/resources/xxxx/\*-dataparam.xml中：

```
<dataparam id="TEST_PARAM_XX" name="测试参数">
        <type>DATAAPP</type>
        <value>select name as label, code as code from MA_BASE_DATA_SET</value>
        <valueScope>NONE</valueScope>
        <dataType>String</dataType>    
        <dataSource>macula_ds</dataSource>
</dataparam>
```

数据参数的表达式可以是SQL语句也可以是静态数据:

* SQL：必须返回code和label两个属性见上述示例，具体语法可以参考数据集的SQL语句编写

* 静态数据：NONE:不缓存\|SESSION:用户Session作用域，用“\|”隔开不同数据，用“：”隔开code和label。如果没有“：”隔开，则code和label一样。


code和label对应着前端下拉框中的code和显示的数据。

## 数据集（DataSet）

### 数据集表达式

表达式可以是一个SQL语句，也可以是包含了Freemarker、Spring EL等表达式的混搭模式的字符串，表达式的内容是任意的，只要下面提到的处理器能处理。需要注意的事DataSet的目的是为了提供一个数据集合，所以表达式往往是一个SQL Select语句。

在特殊情况下，数据集是静态的Key-Value集合，此时数据集可以不需要引用数据源。

```
name:Wilson|name:Jokeway
```

形成的最终数据集为：

```java
List<Object[]> {
       Object[] { "name", "Wilson"},
       Object[] { "name", "Jokeway"}
     }
```

表达式还可以写Freemarker的片段，完全遵循Freemarker的语法。由于在DataSet处理器中，总是传入了UserContext上下文，所以在Freemarker表达式中，默认可以使用的有user变量，以及UserContext中可被解析的变量，均可在Freemarker表达式中解析。

例如，UserContext中的user为{name: Jokeway}，并且能够解析Country属性为String\("China"\)，那么下面的表达式：

```
我的名字是${user.name}
     <#if xyz == 'China'>
      ，你好来自中国的朋友！
     </#if>
```

将被最终解析为：

```
我的名字是Jokeway
     ，你好来自中国的朋友！
```

Macula的DataSet表达式基于Spring EL，并在Spring EL的基础上加入了自定义部分，具体如下：

* \#\(表达式\)\#

  这种写法，Macula认为需要将表达式解析为动态的一个变量替换（如：ptoken1\)，并将表达式的结果与该变量名放入到UserContext的Map中，形成{ptoken1: 表达式的值}的方式。

  _**注意**_

  _一般来讲，这类方式用来处理SQL语句中的条件部分，可以通过JDBC的setParameter方式来设置参数值的情况下使用。_

  * \#\[表达式\]\#

  这种写法，Macula认为需要将表达式解析为一个字符串。

  例如，假如UserContext中能将China变量解析为"中国"，那么：

  ```
  我是#[China]#人
  ```

  将被解析为：

  ```
  我是中国人
  ```

  如果表达式解析出来是一个集合，则将集合中的元素以逗号分隔的形式展开（主要是基于SQL的特点才这样设计），还是上面的例子，如果China解析出来为`List<Integer>{1,2,3}，那么整个结果将被解析为：`

  ```
  我是1,2,3人
  ```

* \#\['表达式'\]\#

  Macula之所以提供这个表达式，主要用于SQL语句解析时，使用IN条件下，需要为表达式中的每个数据都增加引号的情况。

  还是上面的例子，如果使用：

  ```
  我是#['China']#人
  ```

  将被解析为：

  ```
  我是'中国'人
  ```

  和

  ```
  我是'1','2','3'人
  ```

  _**注意**_

  _这种方式在SQL中IN操作情况下特别有用。_


_**重要**_

\#\(\) \#\[\] 中的表达式支持Spring EL的写法，也就是说可以用到下面章节介绍的表达式解析内容。

_DataSet所处理过的字符串均经过了SQL的过滤处理，即会将'替换为''，以避免SQL注入的风险。当然，为了尽可能的避免SQL注入风险，在可以使用\#\(\)\#的地方，不要使用\#\[\]\#。_

### 数据集参数（DataArg）

为了显式标识DataSet详细需要的参数（表达式中使用到的变量），可以设置数据集所需要的参数（DataArg），它主要有以下几个作用：

1. 显式标注变量的数据类型；
2. 可用于将界面输入的条件转化为具体所需的数据类型；
3. 可以设置参数的缺省值。

4. 如果使用DataSetUtils，则dataArg一定要配置，参数通过params参数传入，如果不定义dataArgs，则params参数无效，需要把参数写入userContext


### 数据集的定义方式

数据集（包括数据源）有多种方式可以配置，这是由DataLoader接口的实现去独立完成从各指定的位置获取的。当前大致有两种方式：

1. 基于数据库定义

   用户可通过维护界面（Plugins项目将会提供管理界面）来对数据集进行维护，其实质内容是在数据库层面增加DataSet表的记录，从而达到DataSet的定义目的。

2. 基于XML定义

   考虑到数据集定义的可移植性，以及各系统间同步时的便捷性，增加了DataSet的XML定义方式。其定义模型参考了Spring的Bean定义模型，通过增加Spring的Bean Handler处理以及Schema的限制，实现DataSet的定义。

   在XML中定义的DataSet，其载入方式与Spring ApplicationContext初始化方式一致，即每个DataSet即为一个Spring Bean。由于DataSet的数量众多，以及为了使应用的服务Bean与DataSet分开，DataSet的XML定义将遵循相应的命名规则一致载入。XML文件的命名规则为src/main/resources/data/macula-base/XXX-dataset.xml：

   ```
   <dataset id="TEST_XML_DATA_SET_CODE" name="XML配置DataSet测试">
           <expressionText>select * from MA_BASE_DATA_SET where code=#(code)#</expressionText>
           <pagable>true</pagable>
           <dataSource>macula_ds</dataSource>
           <dataArgs>
               <dataArg label="代码" name="code">
                   <dataType>String</dataType>
                   <fieldControl>Text</fieldControl>
                   <dataParam>TEST_PARAM_XX</dataParam>
               </dataArg>
           </dataArgs>
   </dataset>

   <dataset id="TEST_XML_DATA_SET_CODE2" name="XML配置DataSet测试2">
           <expressionText>select * from MA_BASE_DATA_SET where code=#(code)#</expressionText>
           <dataSource>macula_ds</dataSource>
           <dataArgs>
               <dataArg label="代码" name="code">
                   <dataType>String</dataType>
                   <fieldControl>Text</fieldControl>
               </dataArg>
           </dataArgs>
   </dataset>
   ```


## 数据载入方式

为了统一数据的载入方式，以及方便的定制以及扩展，Macula通过抽象载入接口，可通过实现该接口实现其他方式的载入。

```java
public interface DataLoader<T> extends Ordered {
    /**
     * 根据code加载Data
     * @param code 代码
     * @return T
     */
    public T loader(String code);

    /**
     * 将缓存的data清除
     */
    public void refresh();
}
```

```
public interface DataSetLoader extends DataLoader<DataSet> {

}
```

上节提到的数据库载入以及XML载入就是通过DataSetLoader的两个实现类DbDataSetLoaderImpl和XmlDataSetLoaderImpl完成的。

_**重要**_

_与DataSet相同，数据源（DataSource）以及数据参数（DataParam）都采用了类似的定义和加载方式。_

## DataSetUtils

为了在API层面使用DataSet，macula提供了DataSetUtils类，以便可以获取所定义的DataSet数据集或者数据参数。

