---
type: docs
title: "版本发布"
linkTitle: "版本发布"
weight: 4
description: "本文介绍了版本发布记录"
---

## v3.0.0

* ### Spring 4

  Macula 3 将底层依赖的Spring 升级到Spring 4，以提供对Spring 新特性的支持。Spring 4 支持Java 8， 在核心容器、Web、WebSocket编程、测试等方面都有所提高。

* ### 全新的用户界面

  Macula 3 基于jQuery和Bootstrap构建了一套UI开发框架Mower，并且制定了后端与 前端的标准布局模板，减轻了开发人员构建用户界面的成本，有利于用户体验的提升。

* ### 基于Redis的会话管理

  由于现在的应用系统大多有多个实例，传统的J2EE应用大多基于本JVM管理HTTP会话，但是多个实例之间 会话的同步与管理非常麻烦，Macula 3 基于Redis共享会话，大大简化了集群间用户会话的管理工作。

* ### 微服务框架

  Macula 3 引入开源分布式服务框架Dubbo作为基础，以插件化形式提供对微服务架构的支持，简单易用。

* ### CAT插件

  CAT 是一套基于Java的开源实时应用监控平台，主要应用于服务中间件框架的监控，为开发和运维提供各项性能指标、健康检查、自动报警等可视化服务。Macula提供CAT插件以减少开发人员的埋点工作量。




