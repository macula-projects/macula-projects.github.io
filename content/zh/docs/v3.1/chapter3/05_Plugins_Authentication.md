---
type: docs
title: "用户认证"
linkTitle: "用户认证"
weight: 5
description: "本文介绍了用户认证相关的知识"
---

### 表单登录

### 统一登录

### 预验证登录

如果你的项目需要通过第三方认证系统认证，则可以通过实现PreAuthenticationHandler接口，将第三方认证系统已经验证好的用户名传递给Macula的认证框架实现登录。具体实现如下：

```java
  /**
     * 返回验证好的凭据，一般是用户名，通常是通过第三方认证系统来验证凭据
     * @param request
     * @return UserPrincipal
     */
    public Object getPreAuthenticatedPrincipal(HttpServletRequest request) {
        // 获取用户名和密码，向统一认证验证你的用户名和密码
        // 成功返回用户名，否则返回null;
    }

    /**
     * 通常是密码
     * @param request
     * @return Credentials
     */
    public Object getPreAuthenticatedCredentials(HttpServletRequest request) {
        return “N/A”;
    }
```

### 移动端登录

如果你的项目不愿意跳转到uim实现统一登录，只是需要UIM帮你验证表单上的用户名和密码，则可以通过



