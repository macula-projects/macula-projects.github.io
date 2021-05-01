---
type: docs
title: "OpenAPI"
linkTitle: "OpenAPI"
weight: 10
description: "本文介绍OpenAPI协议"
---

为了更好的提供集成功能，Macula框架可以很方便的提供给第三方调用的接口API，在展示层一章我们已经介绍了怎样将一个普通的Controller方法变成一个开放平台可以调用的API。本章主要介绍第三方调用开放平台API时需要注意的事项。

## Open API的调用

Open API采用JAX-RS标准，所有访问基于HTTP请求进行，Open API的调用参数分为系统级参数和应用级参数。Macula Plugins ESB模块实现了一个OpenApiTemplate，可以使用。

## Open API参数

* 系统级参数：附加在Open API的URL之后，作为Query String传递

| 名称 | 类型 | 是否必要 | 说明 |
| :--- | :---: | :---: | :--- |
| app\_key | String | Y | 在Macula后台中创建应用时分配的KEY |
| timestamp | String | Y | 时间戳，为现在的GMT时间到GMT1970年1月1日0时0分0秒的毫秒数。Open API服务端允许客户端请求时间误差为5分钟。 |
| sign | String | Y | 对API调用的所有参数签名，包括应用级参数 |
| session，macula3不建议使用了，请用access\_token代替 | String | N | CAS的PT，如果是后台应用的调用接口，无需该参数，平台会使用appKey作为用户名判定应用的权限。如果有的话，则会根据PT解析出具体操作的用户，按照具体用户的权限评定是否可以访问该Open API。 |
| access\_token | String | N | 访问UIM的/oauth20/accessToken接口可以获得 |
| format | String | N | 目前支持xml，json格式，默认是json |
| v | String | N | API协议版本号，默认是1.0 |
| sign\_method | String | N | 签名算法，默认支持md5和hmac两种，不写就是md5 |

* 应用级参数：根据规定的请求方式不同，应用级参数传递的方式不同，GET方式应用级参数附加在Open API的URL之后作为Query String传递，POST方式的应用级参数需使用FORM提交的方式传递。根据具体Open API的接口描述，输入参数即为应用级参数，根据输入参数类型的不同，参数名称需要按照下面的规则形成，可以参考展示层FormBean的绑定示例：

  * 原子类型：例如long,int,String等，需要形成“参数名=值”的键值对传递；

  * POJO对象：如User，需要形成“参数名.属性名=值”的键值对传递；

  * 对于POJO对象中如果含有POJO数组、Map、POJO则规则同上，而原子类型数组需要写成“属性名\[index\]=值”。

  * Map&lt;String, POJO&gt;：需要形成“参数名\[key\].属性名=值”的键值对传递；

  * Map&lt;String, String&gt;：需要形成“参数名\[key\]=值”的键值对传递；

  * POJO对象数组：如User\[\]、List&lt;User&gt;，需要形成“参数名\[index\].属性名=值”等的键值对，index为数组下标；

  * 原子类型数组：如String\[\]、List&lt;String&gt;，需要形成“参数名=值0”、“参数名=值1”等的键值对传递；



## OpenAPI参数及签名规则

* 所有的请求和响应数据编码皆为utf-8格式，url里的所有参数值请做urlencode编码。如果请求的Content-Type是application/x-www-form-urlencoded， http body里的所有参数值也做urlencode编码；如果是multipart/form-data格式，每个表单字段的参数值无需编码,但每个表单字段的charset部分需要指定为utf-8。

* 所有api请求内的日期格式都为ISO8601标准，如yyyy-MM-dd'T'HH:mm:ss.SSS'Z'，注意小时格式是24小时制，例如：2008-03-12T18:23:43.233Z。响应内的日期格式和返回格式相同。

* 所有api请求参数内的format\(即返回格式\)可选值为json,xml,默认json。（暂时只支持JSON）

* 签名方式为 md5\(appsecret + key + value .... key + value+appsecret\)然后转大写字母,其中key,value对是除签名和图片外的所有请求参数按key做的升序排列, value无需编码。appsecret是应用注册时系统给出的密钥。hmac的签名方式是hmac\(key+value...+key+value, appsecret\)

* 请注意API的请求方式，非指定方式API不响应。


## Open API的返回

Open API的返回分为正常返回和异常返回。

* 正常返回ExecuteResponse&lt;User&gt;则JSON格式如下：

  ```
  {  
    /** 是否成功标识 */  
    "success" : true,  
    /** User对象 */  
    "returnObject" : {  
    "userName" : "xxx",  
    "password" : "xxx",  
    "org" : {  
      "code" : "xxxx"  
    },  
    "orgs" : [{"code" : "xx"},{"code" : "yy"}],  
    "params" : {  
      "key" : "value"  
    },                      
    "girls" : {                      
      "key" : {"code" : "xx"} },                      
      "date" : "2011-07-11T18:12:35.900Z"                      
    }                      
  }
  ```

* 正常返回PageResponse&lt;User&gt;则JSON格式如下：


```
{
    /** 是否成功标识 */
    "success" : true,
    /** 本次请求的页大小 */
    "size" : 20,
    /** 当前页码，从零开始 */
    "number" : 5,
    /** 总记录数 */
    "totalElements" : 501,
    /** 总页数 */
    "totalPages" : 26,
    /** 本页的总记录数 */
    "numberOfElements" : 20,
    /** 是否首页 */
    "firstPage" : false,
    /** 是否最后页 */
    "lastPage" : false,
    /** 内容列表 */
    "content" : [ {user1 json}, {user2 json}, ... ]
}
```

* 异常返回时的JSON格式如下：

```
{
    /** 是否成功标识 */
    "success" : false,
    /** 系统级错误代码 */
    "errorCode" : "params.10",
    /** 系统级错误信息 */
    "errorMessage" : "缺少APP KEY参数",
    /** 业务级错误代码(如果存在则指的是具体的业务错误信息) */
    "exceptionCode" : "xxx",
    /** 业务级错误信息(如果存在则指的是具体的业务错误信息) */
    "exceptionMessage" : "中文",
    /** 异常详细信息 */
    "exceptionStack" : "error stack"
}
```

在正常返回数据时，如果有警告或提示信息，则上述正常返回的数据中也会含有类似异常返回的数据字段。

## Open API的配置

Open API在调用服务端的验证过程与用户登录方式类似。所以服务端需要做如下配置：  
1. 在应用管理界面中，增加appKey为应用编码的应用配置，并设置密钥为调用时的密钥值；  
2. 在用户表中创建appKey为登录用户名的用户；  
3. 授权该用户具备访问对应地址的权限。

如果是使用access\_token访问，权限与该用户的一致。

## OpenApiTemplate

### 返回值

响应类型基类：

```java
public class Response {
/** 是否成功标识 */
private boolean success;

/** 系统级错误代码 */
private String errorCode;
/** 系统级错误信息 */
private String errorMessage;

/** 业务级错误代码 */
private String exceptionCode;
/** 业务级错误信息 */
private String exceptionMessage;

/** 异常详细信息 */
private String exceptionStack;

/** 服务端重定向信息 */
private String redirection;

/** 校验结果信息 */
private List<FieldError> validateErrors;
}
```

单结果集响应

```java
public class ExecuteResponse<T> extends Response {
/** 结果信息 */
private T returnObject;
}
```

多行结果集响应

```java
public class PageResponse<T> extends Response {
/** 本次请求的记录数 */
private int size;
/** 当前页码，从零开始 */
private int number;
/** 总记录数 */
private long totalElements;
/** 总页数 */
private int totalPages;
/** 本页的总记录数 */
private int numberOfElements;
/** 是否首页 */
private boolean firstPage;
/** 是否最后页 */
private boolean lastPage;
/** 内容列表 */
private List<T> content;
}
```

字段错误类型

```java
public class FieldError {
// 元素名，与页面元素名一致
private String element;
// 错误信息
private String message;
}
```

### 输入

通用条件输入类型

```java
public class CommonCondition {
/** 要查询的条件字段(或属性)名称 */
private String name;
/** 字段数据类型：Boolean, Integer, Long, Double, String, Timestamp, Date */
private DataType dataType;
/**
* 比较符：StartWith, EndWith, Contains, NotContains, Equals, GreaterThan, GreaterOrEqual
* LessThan, LessOrEqual, NotEqual, BeforeThan, AfterThan, Between, Is, In
**/
private CriteriaType criteriaType;
/** 条件值 */
private Object value;
/** 另一个条件值 */
private Object anotherValue;
}

public enum DataType {
  Boolean(Boolean.class), Integer(Integer.class), Long(Long.class),
  Double(Double.class), String(String.class), Timestamp(Timestamp.class), Date(Date.class);
}

public enum CriteriaType {
// like '%x'
StartWith,
// like 'x%'
EndWith,
// like '%x%'
Contains,
// not like '%x%'
NotContains,
// = x
Equals,
// > x
GreaterThan,
// >= x
GreaterOrEqual,
// < x
LessThan,
// <= x
LessOrEqual,
// <> x
NotEqual,
// < x 早于
BeforeThan,
// > x 晚于
AfterThan,
// >= x1 and < x2
Between,
Is,
// in ( x1, x2 )
In ;
}
```

## 



