---
type: docs
title: "安全相关"
linkTitle: "安全相关"
weight: 6
description: "本文介绍安全相关的使用方式"
---

### 跨站伪造请求攻击防护\(CSRF\)

macula框架由于采用了spring-security作为安全框架，所以直接采用了spring-security提供的CsrfFilter，可以通过macula.enableCsrf=true开启。

* 开启CSRF后，macula框架默认会处理AJAX请求的CSRF Token，放入头中；
* 如果不是AJAX提交，则需要自己在表单中加入下面的代码：

  ```
  <#if _csrf?exists>
   <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
  </#if>
  ```

  具体可以参考[Spring Security Reference](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf)

### 跨站脚本攻击防护\(XSS\)

TODO

### 越权访问防护

#### **需要进行防止越权访问的Controller方法改造:**

在需要进行校验的Controller方法上添加@PassKey注解，@PassKey可以支持@PathVariable和@RequestParam。

@PathVariable例子：

```java
@RequestMapping(value ="/deal/edit/{dealId}", method = RequestMethod.GET)
@OpenApi
@PassKey("dealId")
public Long editDealId(@PathVariable("dealId") LongdealId) {
        …
}
```

@RequestParam例子：

```java
@RequestMapping(value ="/deal/edit", method = RequestMethod.GET)
@OpenApi
@PassKey({"dealId"})
public Long editDealId(@RequestParam("dealId") LongdealId) {
…
}
```

如果需要对多个参数值进行校验，可以传入参数名数组，例如：

```
@PassKey({"dealId", "storeId"})
```

#### **产生访问链接的方法改造**

Controller从Service获取domain list后，需要调用PassKeyHelper. generatePassKeyMap方法。方法详情如下：

```java
/**
  *根据传入参数生成passKey map
  *@param list-从数据库获取的domain list
  *@param params-需要校验的参数的参数名
  *@param userName-用户名
  *@return Map<String, String>-<passKeyId(由params拼接，例如需要校验的参数名为id和name，id值为1，name值为abc,
                                                                则passKeyId就为1abc), passkey)
*/
public static Map<String, String> generatePassKeyMap(List<?extendsObject> list, String[] params, String userName)
                                                                throws MaculaException
```

例如：

```java
Map<String, String> map = PassKeyHelper.generatePassKeyMap(dataSourceManagerService.getAllDataSources(), 
                                new String[]{"id", “code”}, 
                                SecurityUtils.getUserDetails().getName());
```

（亦可不传入userName，不传入则交由generatePassKeyMap方法调用SecurityUtils.getUserDetails\(\).getName\(\)获取）

**ftl改造：**

然后在freemarker遍历list的时候，用相应的字段值拼接出passKeyId\(例如需要校验的参数名为id和name，id值为1，name值为abc,则passKeyId就为1abc\)，然后在passKeyMap中取出相应的passKey拼接在URL的后面。

例如：

@PathVariable例子：

```
/deal/edit/123?passKey=255A626422674E0765F51E9C8826B1A3
```

@RequestParam例子：

```
/deal/edit?dealId=123&passKey=255A626422674E0765F51E9C8826B1A3
```

#### 密钥配置

可以在macula.properties:指定加密串，如不指定，将使用macula指定的默认值，例如：

```
pass.key.secret=djij*7dLjdKs20Kds
```

#### **错误提示**

当有人对参数进行篡改，如果是Ajax请求将会返回以下错误：

```
{"success":false,"httpStatus":null,"errorCode":"passkey",
    "errorMessage":"passkey","exceptionCode":"passkey.invalid",
    "exceptionMessage":"不允许平行越权访问","exceptionStack":"…}
```

非Ajax请求抛出异常，由error.ftl处理，默认将会得到以下页面：

#### ![](/images/chapter3/passkey_error.png)

#### **例子**

* 产生domain list的controller方法（这里为了展示多个参数的使用方法，用了id和code两个参数）：

```java
@RequestMapping(value ="/datasource/list", method = RequestMethod.GET)
public String list(Model model) {
    model.addAttribute("datasourceList",dataSourceManagerService.getAllDataSources());
    Map<String, String>passKeyMap= PassKeyHelper.generatePassKeyMap(dataSourceManagerService.getAllDataSources(),
                                                                        newString[] {"id","code"});
    model.addAttribute("passKeyMap",passKeyMap);

    return super.getRelativePath("/datasource/list");
}
```

* 模板页面ftl

```html
<div id="list-${code}" style="width: 100%;">
    <table class="treeTable gridlist">
        <thead>
            <th>数据源编码</th>
            <th>数据源名称</th>
            <th>数据源类型</th>
            <th>访问地址</th>
        </thead>
        <tbody>
            <#list datasourceList as ds>
                <#assign passKeyId=ds.id+ds.code />
                <tr>
                    <td>${ds.code}</td>
                    <td>${ds.name}</td>
                    <td>${ds.dataSourceType}</td>
                    <td><a href="${base}/admin/macula-base/datasource/edit?id=${ds.id}&code=${ds.code}&passKey=${passKeyMap[passKeyId]}">点击访问</a>
                </tr>
            </#list>
        </tbody>
    </table>
</div>
```

* 需要防止越权访问的controller方法：

```java
@RequestMapping(value ="/datasource/edit", method = RequestMethod.GET)
@PassKey({"id","code"})
public String edit(@RequestParam("id") Long id, @RequestParam("code") String code, Model model) {
    model.addAttribute("id", id);
    return super.getRelativePath("/datasource/edit");
}
```



