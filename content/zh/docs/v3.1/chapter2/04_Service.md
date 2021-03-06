---
type: docs
title: "业务服务层"
linkTitle: "业务服务层"
weight: 5
description: "本文介绍业务服务层的开发方式"
---

Macula开发平台自身是基于Spring作为开发基础，在Spring Framework的基础上，增加的一些规则和CoC的标准与规范，所以对于服务层，更多的需要介绍的是服务层的开发规范。

业务服务层通过Spring Bean来定义，其开发本身与一般的基于Spring的J2EE开发一致，下面就Macula平台所做出的一些要求做一些介绍。

## 业务服务层必须具备易读性接口

业务服务层涉及的代码量在整个开发中占据非常大的比例，为了保证程序代码的整洁和易读性，对服务层的方法都必须按功能、用途、数据访问情况等方式，分解到一个或多个接口中，并在命名上具备相当的可读性，使代码阅读者能通过接口名称以及方法名称直观的了解到业务方法的目的。

## 业务服务层必须对参数进行校验

业务服务层最总的调用者可能来自Controller层，也可能来自WebService层，所以对于服务层方法的输入参数，需要进行合法性检查，可通过assert\(boolean, message\)等助手方法，尽快将不合法的输入参数以异常的方式抛出，而不是在程序边运行边检查，这样容易造成开销的增大，同时也失去了结构的严谨性以及代码的可读性。

## 服务层的事务配置

对于服务层的事务注解，将放在实现类的方法定义上，需要注意的是，需要事务注解的方法，只能是接口中已经定义的方法，同时需要注意根据实际的应用情况，确定事务的只读性，比如查询类的方法，可增加事务的readOnly=true标记，以增强系统的运行效率。

## 服务层的Spring Bean定义

如无特殊的需要，尽量使用@Service来定义业务层的Bean，对于主要注入的其他依赖的Bean，则通过构造函数的方式注入，通过构造函数的注入方式是Spring 3.0后推崇的做法，具体可参考Spring Framework相关文档。

### 服务层的异常

服务层主要是调用Repository，处理事务，这一层不需要特殊处理异常，如果异常会影响你的业务逻辑，则需要try然后按照您的逻辑编写代码，如果是一些校验类的异常，可以直接抛出。ServiceExceptionHandler会采用AOP方式统一处理@Service注解的服务层类。具体可以参考异常处理一节。

