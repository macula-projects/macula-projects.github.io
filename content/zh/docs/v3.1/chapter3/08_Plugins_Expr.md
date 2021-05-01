---
type: docs
title: "表达式解析"
linkTitle: "表达式解析"
weight: 8
description: "本文介绍macula支持的表达式"
---

## 表达式说明

macula使用标准的Spring EL表达式，具体语法可以参考。macula支持表达式的功能有

* 用户规则定义的规则表达式

* 数据集、数据参数的SQL语句（\#\(\) \#\[\]\# 里面可以用表达式\)


默认情况下，macula表达式可以使用的变量有：

* user：代表是当前用户的userPrincipal
* 系统参数：为参数表中定义的内容，引用时使用DataParam$前缀加DataParamCode的方式。
* 自定义参数：通过new UserContextWrapper\(userContext, params\)添加自定义参数。
* 实现ValueEntryResolver接口扩展表达式。

## 实现ValueEntryResolver接口

可以通过实现ValueEntryResolver接口，使得表达式中可以直接使用该接口实现对应的KEY，也可以通过UserContext.resolve\(\)方法获取该接口实现提供的数据。

```java
public interface ValueEntryResolver extends Ordered, Comparable<ValueEntryResolver> {

    /**
     * 是否能解析.
     */
    boolean support(String key);

    /**
     * 解析指定的值.
     */
    ValueEntry resolve(String attribute, UserContext userContext);

}
```

需要注意的是，为了提高resolve接口对数据的解析速度，可以在resolve的实现中，对返回的ValueEntry设置合理的缓存级别。

## 表达式解析

表达式的解析都是通过UserContextImpl中的如下方法：

```
@Override
public EvaluationContext createEvaluationContext() {
        EvaluationContext evalutionContext = new StandardEvaluationContext(this);
        // 可以通过user.ORG等获取数据，具体是调用user.getCatlogCodes('xxx')
        evalutionContext.getPropertyAccessors().add(
                new CustomMethodPropertyAccessor(UserPrincipal.class, "getCatalogCodes", UserContextStaticServiceHolder
                        .getSecurityCatalogService().getAvaliableProviderNames()));
        // 可以通过user.MENU等获取数据，具体是调用user.getResourceCodes('xxx')
        evalutionContext.getPropertyAccessors().add(
                new CustomMethodPropertyAccessor(UserPrincipal.class, "getResourceCodes",
                        UserContextStaticServiceHolder.getSecurityResourceService().getAvaliableProviderNames()));
        // 不能识别的属性会调用resolve
        evalutionContext.getPropertyAccessors().add(
                new CustomMethodPropertyAccessor(UserContext.class, "resolve", null));
        return evalutionContext;
}
```

EvaluationContext是Spring EL解析表达式的上下文，Spring EL表达式解析过程如下：

```
ExpressionParser parser = new SpelExpressionParser();
org.springframework.expression.Expression exp = parser.parseExpression(expressionText());
return exp.getValue(EvaluationContext, Result.class);
```

在需要解析表达式的地方都是如上述代码处理的。所以UserContext中的getXXX\(\)都是可以作为表达式中的变量的，比如getUser\(\)方法可以在表达式中直接写成 user，获取用户名可以写表达式“user.name"，user就是UserPrincipal。

上述evaluationContext还注册了：

* user.ORG等价于user.getCatlogCodes\('xxx'\)；


* user.MENU等价于user.getResourceCodes\('xxx'\)；

* 不能识别的属性会调用UserContext中的resolve方法，该方法调用ValueEntryResolver接口的实现去解析表达式。