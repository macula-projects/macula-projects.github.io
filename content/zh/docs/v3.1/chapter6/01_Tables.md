---
type: docs
title: "表结构"
linkTitle: "表结构"
weight: 1
description: "本文介绍macula表结构"
---

## 应用管理

### MA_BASE_APPLICATION

**表 23.1. MA_BASE_APPLICATION**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主关键字 |
| APP_ID | 应用的ID | VARCHAR2(20) | NOT NULL |  | 唯一索引 |
| APP_NAME | 应用名称 | VARCHAR2(50) | NOT NULL |  |  |
| HOME_PAGE | 应用入口地址 | VARCHAR2(255) | NOT NULL |  |  |
| SECURE_KEY | 应用公钥 | VARCHAR2(1024) | NOT NULL | | 	OpenAPI访问时的密码 |
| PRIVATE_KEY | 应用私钥 | VARCHAR2(1024) | NOT NULL | 保留 |  |
| CONTACT | 联系方式 | VARCHAR2(255) | NULL | | |
| SUPERVISOR | 应用负责人 | VARCHAR2(255) | NULL |  |  |
| IS_SSIN | 是否支持单点登录 | NUMBER(1) | NOT NULL | 默认:1 | |
| IS_SSOUT | 是否支持单点登出 | NUMBER(1) | NOT NULL | 默认:1 | |
| THEME | 界面风格 | VARCHAR2(20) | NULL | | |
| USE_ATTRS | 是否回传属性 | NUMBER(1) | NOT NULL | 默认:0 | |
| ALLOWED_ATTRS | 回传属性 | VARCHAR2(1024) | NULL |  |  |
| COMMENTS | 应用备注 | VARCHAR2(255) | NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  |  &nbsp; |

### MA_BASE_APP_INSTANCE

**表 23.2. MA_BASE_APP_INSTANCE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主关键字 |
| APP_ID | 应用的ID | VARCHAR2(20) | NOT NULL |  | 外键 |
| CODE | 实例代码 | VARCHAR2(50) | NOT NULL |  | APP_ID+CODE唯一索引 |
| NAME | 实例名称 | VARCHAR2(50) | 	NOT NULL |  |  |
| HOME_PAGE | 实例入口地址 | VARCHAR2(255) | NOT NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

## 数据管理

### MA_BASE_DATA_SOURCE

**表 23.3. MA_BASE_DATA_SOURCE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 数据源代码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 数据源名称 | VARCHAR2(50) | NOT NULL |  |  |
| TYPE | 数据源类型 | VARCHAR2(10) | NOT NULL | 默认:DATABASE | 目前支持DATABASE和LDAP两种类型 |
| IS_JNDI | 是否使用jndi | NUMBER(1) | NOT NULL | 默认:0 |  |
| URI | JDBC URL地址 | VARCHAR2(255) | NOT NULL |  | 如果是JNDI，这里填写JNDI NAME |
| DRIVER | JDBC驱动 | VARCHAR2(255) | NULL |  |  |
| USER_NAME | 数据库用户名 | VARCHAR2(50) | NULL|  |  |
| PASSWORD | 数据库密码 | VARCHAR2(50) | NULL |  |  |
| VALIDATION_QUERY | 验证语句 | VARCHAR2(255) | NULL |  |  |
| MAX_ACTIVE | 最大活动时间 | NUMBER(10) | NOT NULL | 默认:0 |  |
| MAX_IDLE | 最大空闲连接数 | NUMBER(10) | NOT NULL | 默认:0 |  |
| MAX_SIZE | 最大连接数 | NUMBER(10) | NOT NULL | 默认:0 |  |
| MAX_WAIT | 最大等待时间 | NUMBER(10) | NOT NULL | 默认:0 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_DATA_ENUM

**表 23.4. MA_BASE_DATA_ENUM**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| TYPE | 枚举类型 | VARCHAR2(10) | NOT NULL |  | 唯一索引 |
| CODE | 枚举编码 | VARCHAR2(50) | NOT NULL |  |  |
| NAME | 枚举名称 | VARCHAR2(50) | NOT NULL |  |  |
| LOCALE | 枚举语言 | VARCHAR2(255) | NOT NULL |  | 按照java.util.Locale的标准 |
| ORDERED | 排序 | NUMBER(10) | NOT NULL | 默认:0 |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| IS_ENABLED | 是否有效 | NUMBER(1) | NOT NULL | 默认:1 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  |  |
| COMMENTS | 备注 | VARCHAR2(255) | NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_DATA_PARAM

**表 23.5. MA_BASE_DATA_PARAM**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| TYPE | 参数类型 | VARCHAR2(10) | NOT NULL |  | 唯一索引 |
| CODE | 参数代码 | VARCHAR2(50) | NOT NULL |  |  |
| NAME | 参数名称 | VARCHAR2(50) | NOT NULL |  |  |
| VALUE | 参数值 | 	VARCHAR2(500) | NOT NULL |  |  |
| VALUE_SCOPE | 参数值缓存级别 | VARCHAR2(20) | NOT NULL |  | NONE:不缓存 SESSION:会话级 INSTANCE:实例级 APPLICATION:应用级 |
| PARAM_CLZ | 数据类型 | VARCHAR2(10) | NULL |  | Boolean、Integer、 Long、Double、 String、Timestamp、 Date |
| DATASOURCE_ID | 数据源ID | NUMBER(19) | NULL |  | 外键 |
| ORDERED | 排序 | 	NUMBER(10) | NOT NULL | 默认:0 |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| IS_ENABLED | 是否有效 | NUMBER(1) | NOT NULL | 默认:1 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL | 外键 |  |
| COMMENTS | 备注 | VARCHAR2(255)| NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_DATA_SET

**表 23.6. MA_BASE_DATA_SET**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 数据集代码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 数据集名称 | VARCHAR2(50) | NOT NULL |  |  |
| EXP_TEXT | 数据集表达式 | CLOB | NULL |  |  |
| HANDLER_CHAIN | 数据集处理链 | CLOB | NULL |  |  |
| DATASOURCE_ID | 数据源ID | NUMBER(19) | NULL |  | 外键 |
| IS_PAGABLE | 数据集是否分页 | NUMBER(1) | NOT NULL | 默认:1 |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  | 外键 |
| MID | 所属菜单 | NUMBER(19)| NULL |  | 外键 |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL|  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  |  |

### MA_BASE_DATA_ARG

**表 23.7. MA_BASE_DATA_ARG**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 	主键 |
| DATASET_ID | 数据集ID | NUMBER(19) | NOT NULL |  | 外键 |
| ARG_NAME | 输入参数名称 | VARCHAR2(20) | NOT NULL |  | DATASET_ID+ARG_NAME唯一索引 |
| ARG_LABEL | 参数标题 | VARCHAR2(50) | NOT NULL |  | 标题 |
| ARG_CLZ | 输入参数类型 | VARCHAR2(50) | NOT NULL |  | Boolean、Integer、 Long、Double、 String、Timestamp、 Date |
| ARG_CONTROL | 参数控件 | VARCHAR2(20) | NOT NULL | Text | 枚举 Text等 |
| DEFAULT_VALUE | 缺省值 | VARCHAR2(50) | NULL |  |  |
| DATAPARAM_ID | 数据参数ID | NUMBER(19) | NULL |  | 外键 |
| ALLOW_NULL | 	是否允许为空 | NUMBER(1) | NOT NULL| 默认:0 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

## 资源和分组

### MA_BASE_ACL_MENU

**表 23.8. MA_BASE_ACL_MENU**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID	 | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| CODE | 菜单编码 | VARCHAR2(50)| NOT NULL |  | 唯一索引 |
| NAME | 菜单名称 | VARCHAR2(50)| NOT NULL |  |  |
| DESCRIPTION | 菜单描述 | VARCHAR2(255) | 	NULL |  |  |
| URI | ACTION的地址 | VARCHAR2(255) | NULL |  |  |
| HTTP_METHOD | HTTP请求方式 | VARCHAR2(10) | NULL |  | GET，POST |
| LOG_OPTION | 	日志记录选项  | NUMBER(10) | NULL |  | REQUEST_HEAD REQUEST_PARAM SESSION_ATTR REQUEST_ATTR之组合 |
| LOG_LEVEL | 日志记录级别 | VARCHAR2(10) | NULL|  | ANY：全部请求 LOGON：登录后的请求 ERROR：出错后的请求 |
| EFFECTIVE_TIME | 生效日期 | TIMESTAMP(6)| NULL |  |  |
| INACTIVE_TIME | 失效日期 | TIMESTAMP(6)| NULL |  |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  | 外键 |
| ORDERED | 排序序号 | NUMBER(10) | NOT NULL | 默认:0 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 	最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_ORG

**表 23.10. MA_BASE_ACL_ORG**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| OA_ID | OA中的组织ID | NUMBER(19) | NOT NULL |  | 唯一索引 |
| APP_ID | 	应用ID | VARCHAR2(20) | NULL |  |  |
| CODE | 组织编码 | VARCHAR2(50) | NOT NULL|  | 唯一索引 |
| NAME | 组织名称 | VARCHAR2(50) | NOT NULL |  |  |
| SIMPLE_NAME | 组织简称| VARCHAR2(50) | NOT NULL |  |  |
| NICK_NAME | 组织昵称 | VARCHAR2(255) | NULL |  |  |
| PARENT_ID | 父组织ID | NUMBER(19) | NULL | OA的ID | 不建外键 |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:1 |      |
| ORDERED | 排序 | NUMBER(10) | NOT NULL |  |  |
| LEADER_ACCOUNT | 	组织主管帐号 | VARCHAR2(255) | 	NULL |  |  |
| FOUND_DATE | 组织创建日期 | DATE | NOT NULL |  |  |
| ORG_LEVEL | 组织级别 |NUMBER(10)| NOT NULL | 默认:0 |  |
| ORG_TYPE | 组织类型 | VARCHAR2(2) | NOT NULL |  | 'HO'：总部用户；'B0'：分公司用户；'S0'：服务中心用户；'HC'：货仓用户 |
| IS_ASSIGNABLE | 是否可分配 | NUMBER(1) | NOT NULL | 默认:1 |  |
| ENABLED | 是否有效 | NUMBER(1) | NOT NULL | 默认:1 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_PROVIDER_DEF

**表 23.11. MA_BASE_ACL_PROVIDER_DEF**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| APP_ID | 	应用ID | VARCHAR2(20) | NULL |  |  |
| NAME | 名称 | VARCHAR2(20) | NOT NULL |  | 唯一索引 |
| GROUP_NAME | 分类 | VARCHAR2(20) | NOT NULL |  | CATALOG_PROVIDER :分组提供者 RESOURCE_PROVIDER :资源提供者 |
| PROTOCOL | 协议 | VARCHAR2(20) | NOT NULL |  | HESSIAN BURLAP JSON_REST |
| URI | 地址 | VARCHAR2(255) | NULL |  |  |
| USER_NAME | 用户名 | VARCHAR2(50) | NULL |  |  |
| PASSWORD | 密码 | VARCHAR2(50) | NULL | | |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

## 业务策略

### MA_BASE_ACL_BIZ_RULE

**表 23.12. MA_BASE_ACL_BIZ_RULE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 业务规则编码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 业务规则名称 | VARCHAR2(50) | NOT NULL |  |  |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| EXP_TEXT | 业务规则表达式 | CLOB | NULL |  |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  | 外键 |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_USER_RULE

**表 23.13. MA_BASE_ACL_USER_RULE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 用户规则编码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 用户规则名称 | VARCHAR2(50) | NOT NULL |  |  |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| EXP_TEXT | 用户规则表达式 | CLOB | NULL |  |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_POLICY

**表 23.14. MA_BASE_ACL_POLICY**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 业务策略编码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 业务策略名称 | VARCHAR2(50) | NOT NULL |  |  |
| APP_ID | 应用ID | VARCHAR2(20) | NULL |  |  |
| IS_DECISION | 是否决策规则 | NUMBER(1) | NOT NULL | 默认:0 |  |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_POLICY_ENTRY

**表 23.15. MA_BASE_ACL_POLICY_ENTRY**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| GRANTEDPOLICY_ID | 策略ID | NUMBER(19) | NOT NULL |  | 外键 |
| USERRULE_ID | 用户规则ID | NUMBER(19) | NOT NULL |  | 外键 |
| DECISIONRULE_ID | 决策规则ID | NUMBER(19) | NULL |  | 外键 |
| QUERYRULE_ID | 查询规则ID | NUMBER(19) | NULL |  | 外键 |
| DENY_REASON | 拒绝原因 | VARCHAR2(255) | NULL |  |  |
| ORDERED | 排序 | NUMBER(10) | NOT NULL | 默认:0 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

## 用户授权

### MA_BASE_ACL_ROLE

**表 23.16. MA_BASE_ACL_ROLE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| CODE | 角色代码 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| NAME | 角色名称 | VARCHAR2(50) | NOT NULL |  |  |
| IS_ASSIGNABLE | 是否可分配 | NUMBER(1) | 	NOT NULL | 默认:1 |  |
| IS_RESIGNABLE | 是否可下放分配 | NUMBER(1) | NOT NULL | 默认:1 |  |
| IS_RULE | 是否规则角色 | 	NUMBER(1) | NOT NULL | 默认:0 |  |
| IS_INHERITABLE | 是否可继承 | NUMBER(1) | NOT NULL | 默认:1 |  |
| INHERIT_FROM | 继承自 | NUMBER(19) | NULL |  | 外键 |
| IS_EXECUTABLE | 是否可执行 | NUMBER(1) | NOT NULL | 默认:1 |  |
| USERRULE_ID | 用户规则ID | NUMBER(19) | NULL |  | 外键 |
| IS_GROUP | 是否分组 | NUMBER(1) | NOT NULL | 默认:0 |  |
| PARENT_ID | 分组父ID | NUMBER(19) | NULL |  | 外键 |
| IS_OPPOSITE | 是否反向角色 | NUMBER(1) | NOT NULL | 默认:0 |  |
| IS_INHERITABLE | 是否可继承 | NUMBER(1) | NOT NULL | 默认:1 |  |
| INHERIT_FROM | 继承自 | NUMBER(19) | NULL |  |  |
| IS_EXECUTABLE | 是否可执行 | NUMBER(1) | NOT NULL | 默认:1 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_ROLE_RES

**表 23.17. MA_BASE_ACL_ROLE_RES**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| RES_TYPE | 资源类型 | VARCHAR2(10) | NOT NULL |  | 唯一索引 |
| ROLE_ID | 角色ID | NUMBER(19) | NOT NULL |  | 唯一索引 |
| RES_ID | 资源ID | NUMBER(19) | NOT NULL |  | 唯一索引 |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_USER_ROLE

**表 23.18. MA_BASE_ACL_USER_ROLE**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| USER_NAME | 用户名 | VARCHAR2(50) | NOT NULL |  | USER_NAME+ROLE_ID唯一索引 |
| ROLE_ID | 角色ID | NUMBER(19) | NOT NULL |  | 外键 |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |


### MA_BASE_ACL_USER_CAT

**表 23.19. MA_BASE_ACL_USER_CAT**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| USER_NAME | 用户名 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| CAT_TYPE | 分组类型 | VARCHAR2(10) | NOT NULL |  | 唯一索引 |
| CAT_ID | 分组ID | NUMBER(19) | NOT NULL |  | 唯一索引 |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

## 系统信息

### MA_BASE_ACL_USER_SESSION

**表 23.20. MA_BASE_ACL_USER_SESSION**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| USER_NAME | 用户名 | VARCHAR2(50) | NOT NULL |  | 唯一索引 |
| LOGIN_USER_NAME | 登录后的用户名 | VARCHAR2(50) | NULL |  |  |
| IS_LOGIN_IN | 是否在线 | 	NUMBER(1) | NOT NULL | 	默认:0 |  |
| LAST_CHG_PWD_TIME | 上次密码修改时间 | TIMESTAMP | NULL |  |  |
| NEED_CHG_PWD | 登录后是否需要修改密码 | NUMBER(1) | NOT NULL |  |  |
| OLD_PWD | 密码历史 | VARCHAR2(250) | NULL |  | 用逗号分割 |
| LOGIN_FAILED_TIME | 	登录失败时间 | NUMBER(10) | TIMESTAMP |  |  |
| PWD_FAILED_TIMES | 密码输入出错次数 | NUMBER(10) | NOT NULL | 默认:0 |  |
| RETRIEVE_PWD_TIMES | 获取密码次数 | NUMBER(10) | 	NOT NULL | 默认:0 |  |
| RETRIEVE_PWD_TIME | 最后获取密码时间 | TIMESTAMP | NULL |  |  |
| IS_LOCKED | 是否锁定 | NUMBER(1) | NOT NULL | 默认:0 |  |
| LOCKED_TIME | 锁定时间 | TIMESTAMP | NULL |  |  |
| REM_ME_LAST_USED | RememberMe最后使用时间  | TIMESTAMP | 	NULL |  |  |
| REM_ME_SERIES | RememberMe序列号 | VARCHAR2(64) | NULL |  |  |
| REM_ME_TOKEN | RememberMe值 | VARCHAR2(64) | NULL |  |  |
| LAST_LOGIN_ADDR | 最后登录IP |VARCHAR2(255) | NULL |  |  |
| LAST_LOGIN_BROWSER | 最后登录浏览器 | VARCHAR2(255) | NULL |  |  |
| LAST_LOGIN_INST | 最后登录实例| VARCHAR2(255) | NULL |  |  |
| LAST_LOGIN_TIME | 最后登录时间 | TIMESTAMP | NULL |  |  |
| LAST_LOGOUT_TIME | 最后登出时间 | TIMESTAMP | NULL |  |  |
| ILLEGAL_REQUEST | 是否锁屏请求 | 	NUMBER(1) | NOT NULL | 默认值: 0 |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |

### MA_BASE_ACL_USER_SESSION_HIS

**表 23.21. MA_BASE_ACL_USER_SESSION_HIS**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| USER_NAME | 用户名 | VARCHAR2(50) | NOT NULL |  | 索引 |
| EVENT_INSTANCE | 应用实例 | VARCHAR2(255) | NOT NULL |  |  |
| EVENT_ADDR | 请求IP地址 | VARCHAR2(255) | NULL |  |  |
| EVENT_BROWSER | 请求浏览器 | VARCHAR2(255) | NULL |  |  |
| EVENT_TYPE | 事件类型 | VARCHAR2(10) | NOT NULL |  |  |
| EVENT_TIME | 事件时间 | TIMESTAMP | NOT NULL |  |  |
| EVENT_RESULT | 事件结果 | VARCHAR2(10) | NOT NULL |  |  |
| EVENT_COMMENTS | 事件备注 | VARCHAR2(255) | NULL |  | &nbsp;|


### MA_BASE_ACL_ACCESS_LOG

**表 23.22. MA_BASE_ACL_ACCESS_LOG**

| 字段名称 | 中文名称 | 数据库字段类型 | 是否允许为空 | 默认值 | 备注 |
| -- | -- | -- | -- | -- | -- |
| ID | 顺序号 | NUMBER(19) | NOT NULL |  | 主键 |
| USER_NAME | 用户名 | VARCHAR2(255) | NULL |  | 索引 |
| ACTION_ID | 功能ID | NUMBER(19) | NOT NULL |  | 索引 |
| APP_ID | 应用的ID | VARCHAR2(50) | NOT NULL |  |  |
| APP_INST | 应用实例 | VARCHAR2(50) | NOT NULL |  |  |
| START_TIME | 请求开始时间 | TIMESTAMP(6) | NOT NULL |  |  |
| END_TIME | 请求结束时间 | TIMESTAMP(6) | NOT NULL |  |  |
| REQUEST_ADDR | 远程客户端地址 | VARCHAR2(50) | NOT NULL |  |  |
| REQUEST_URL | 请求URL | VARCHAR2(1024) | NOT NULL |  |  |
| REQUEST_DETAIL | 请求明细 | CLOB | NULL |  |  |
| RESPONSE_CODE | 响应CODE | VARCHAR2(10) | NULL |  |  |
| ERROR_MESSAGE | 错误信息 | CLOB |	NULL |  |  |
| CREATED_BY | 创建人 | VARCHAR2(50) | NOT NULL |  |  |
| CREATED_TIME | 创建时间 | TIMESTAMP | NOT NULL |  |  |
| LAST_UPDATED_BY | 最后更新人 | VARCHAR2(50) | NOT NULL |  |  |
| LAST_UPDATED_TIME | 最后更新时间 | TIMESTAMP | NOT NULL |  | &nbsp; |








