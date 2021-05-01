---
type: docs
title: "权限管理"
linkTitle: "权限管理"
weight: 7
description: "本文介绍权限管理相关的原理和设计"
---


权限管理作为macula-base的主要部分，包括了用户属性、用户组、角色、菜单、资源、功能权限、数据权限等多方面的内容。

在用户权限架构方面，围绕用户自身，主要有用户分组和用户角色两大块。

对于资源部分，当前实现的资源主要有菜单资源和功能资源，用来解决功能权限的问题，对于数据权限，将通过与角色、用户属性、待检测数据联合作用的方式来实现数据权限。

## 权限管理的几大主体

权限管理主要是围绕用户、角色、用户组等几方面而形成的对某资源具有操作或不可操作的问题。它包含的主体有：

* 用户（User）

  来自J2EE世界的Principal，对应着登录用户。

* 用户分组（Catalog）

  按一定的规则将用户分成的类别，比如组织机构就是一种用户分组。

  用户分组可按组织机构分（Organization），也可按用户组分（UserGroup）或其他的分类方式。

  用户分组是一种逻辑意义上的用户分类，它主要是将一些信息附属到用户信息上。

* 角色（Role）

  角色是权限授权与鉴权的主体，角色将于用户形成用户角色管理，角色同时与资源也形成角色资源关联。在校验用户是否具备操作某资源时，实际上是通过检测用户角色列表与可操作资源的角色列表之间是否存在交集的问题。

  在Macula权限设计中，角色可分为普通角色与计算角色。

  * 普通角色：是指需要主动与用户产生关联关系后，用户才能拥有的角色；
  * 规则角色：是指不需要主动与用户产生关联，通过用户已有的属性，以及角色给定的表达式，计算用户是否具备的角色。

    在1.1.0版之后，加入了角色可继承属性，继承类型的角色，只能是普通角色，其可授权范围受限于其所继承的角色。

    对于非系统管理员来说，只能创建继承角色。



* 资源（Resource）

  资源是一类具有时间性的可访问或操作的集合，最有代表的资源就是系统的菜单。

* 应用（Application）

  是指具体的业务系统应用，这里只是应用的一个定义。

* 这几个权限主体的相互关系如下图所示：

  ![macula-security-acl.jpg](/imgs/v3.1/chapter3/macula-security-acl.jpg "macula-security-acl.jpg")


## 权限配置

![macula-security-biz.jpg](/imgs/v3.1/chapter3/macula-security-biz.jpg "macula-security-biz.jpg")

## 用户分组提供者接口

默认情况下，Macula框架只提供了基于组织结构的用户分组，如果需要添加其他的用户分组信息，可以通过实现SecurityCatalogProvider的方式提供新的用户分组信息。

Macula框架实现了一个抽象类AbstractCatalogProvider，可以基于AbstractCatalogProvider快速添加一个新的用户分组，不过用户分组与用户之间的关系会统一由Macula框架来管理，如果需要自行管理，则不能使用该抽象类。

```java
public interface SecurityCatalogProvider extends SecurityProvider {

    /**
     * 获取该分类下的所有信息.
     * 
     * @return 该分类所有信息列表
     */
    List<CatalogData> list();

    /**
     * 获取指定用户在该分类下所拥有的分类列表.
     * 
     * @param username
     *            用户名
     * @return 用户在该分类下的信息列表
     */
    List<CatalogData> getCatalogsByUser(String username);

    /**
     * 获取该分类下具体值所关联的用户列表.
     * 
     * @param catalogId
     *            具体的分类值
     * @return 用户列表信息
     */
    List<String> getUsersByCatalog(Long catalogId);

    /**
     * 在分类下加入用户关联信息
     * 
     * @param username
     *            用户名
     * @param catalogIds
     *            具体的分类值列表
     */
    void addCatalogsByUser(String username, Collection<Long> catalogIds);

    /**
     * 在分类下加入用户关联信息
     * 
     * @param usernames
     *            用户名列表
     * @param catalogId
     *            具体的分类值
     */
    void addUsersByCatalog(Collection<String> usernames, Long catalogId);

    /**
     * 在分类下删除用户关联信息
     * 
     * @param username
     *            用户名
     * @param catalogIds
     */
    void removeCatalogsByUser(String username, Collection<Long> catalogIds);

    void removeUsersByCatalog(Collection<String> usernames, Long catalogId);

}
```

## 资源提供者接口

资源是一类具有时间性的可访问或操作的集合，比如系统菜单，用户可以通过实现SecurityResourceProvider接口，来达到通过Macula框架管理您指定资源的目的。实际上可以理解为一个细粒度的数据权限行为。

Macula框架已经实现了一个抽象类AbstractResourceProvider，用于快速注册一个资源到Macula框架，基于AbstractResourceProvider抽象类时，资源与角色的关系由Macula框架自行管理，否则不可以使用AbstractResourceProvider类。

```java
public interface SecurityResourceProvider extends SecurityProvider {

    /**
     * 所有的资源信息列表
     */
    List<ResourceData> list();

    /** 指定角色的关联资源信息列表 */
    List<ResourceData> getResourcesByRole(Long roleId);

    /** 指定角色列表的关联资源信息列表 */
    List<ResourceData> getResourcesSetByRoles(Collection<Long> roleIds);

    /** 获取角色关联资源树 */
    List<ResourceData> getResourcesTreeByRoles(Collection<Long> roleIds, Long root, int level);

    /** 获取角色对应的关联资源 */
    Map<Long, List<ResourceData>> findRoleResourceMap(Collection<Long> roleIds);

    /** 获取指定资源关联的角色列表 */
    List<Long> getRolesByResource(Long resourceId);

    /** 获取资源对应的资源、角色列表映射 */
    Map<Long, List<Long>> findResourceRoleMap(Collection<Long> resourceIds);

    /** 增加角色与资源关联 */
    void addResourcesByRole(Long roleId, Collection<Long> resourceIds);

    /** 增加角色与资源关联 */
    void addRolesByResource(Collection<Long> roleIds, Long resourceId);

    /** 删除角色与资源关联 */
    void removeResourcesByRole(Long roleId, Collection<Long> resourceIds);

    /** 删除角色与资源关联 */
    void removeRolesByResource(Collection<Long> roleIds, Long resourceId);
}
```

## 权限过滤详解

在应用Spring Security来保护整个应用地址的大前提下，Macula相应的对整个Filter链进行了改写和定制，具体如下：

```
<bean id="maculaDefaultSecurityFilterChain" class="org.springframework.security.web.DefaultSecurityFilterChain">
        <constructor-arg index="0">
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg index="0" value="/**" />
            </bean>
        </constructor-arg>
        <constructor-arg index="1">
            <list>
                <ref bean="maculaExceptionNegotiateFilter" />
                <ref bean="maculaAccessLogFilter" />
                <ref bean="maculaApplicationInstanceAgentFilter" />

                <!-- 将认证信息保存到Session中(SecurityContext) -->
                <ref bean="maculaSecurityContextPersistenceFilter" />
                <!-- 检查SessionInformation中有无过期会话，有就强制清除 -->
                <ref bean="maculaConcurrentSessionFilter" />

                <!-- 启动跨站脚本攻击的防护 -->
                <ref bean="maculaCsrfFilter" />

                <!-- 只有登录或者登出才会通过下面的Filter -->
                <!-- CAS退出后会请求所有授权过的服务，这个Filter就是处理单点登出，把token转为session并失效 -->
                <ref bean="maculaSingleSignOutFilter"/>                
                <!-- 退出请求时，SecurityContextLogoutHandler会让session会失效 -->
                <ref bean="maculaLogoutFilter" />
                <!-- 预先通过第三方系统认证用户，该Filter会直接认可这个用户并完成后续的登录处理 -->
                <ref bean="maculaPreAuthenticatedProcessingFilter"/>
                <ref bean="maculaCasAuthenticationFilter" />
                <ref bean="maculaFormAuthenticationFilter" />
                <ref bean="maculaCasRestAuthenticationFilter" />
                <ref bean="maculaOpenApiAuthenticationFilter" />

                <!-- Exception Filter会保存request，这里看看能不能恢复之前异常后保存的请求参数 -->
                <ref bean="maculaRequestCacheAwareFilter" />

                <!--ref bean="maculaRemembermeAuthenticationFilter" /-->
                <!-- Wrapper request，方便提取认证信息getUserPrincipal -->
                <ref bean="maculaSecurityContextHolderAwareFilter" />
                <ref bean="maculaAnonymousAuthenticationFilter" />

                <ref bean="maculaSessionManagementFilter" />

                <!-- Exception配合SecrityFilter，如果通不过安全检查，则通过EntryPoint跳转 -->
                <ref bean="maculaExceptionTranslationFilter" />
                <ref bean="maculaSecurityFilter" />

                <!-- 自定义检查SESSION中的用户与页面请求的是否是同一个用户 -->
                <ref bean="userHeaderIdentityFilter" />
            </list>
        </constructor-arg>
</bean>
```



