---
layout:     post
title:      SpringCloud学习笔记（一）
subtitle:   微服务架构概述
date:       2020-04-10
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringCloud
    - SpringCloud学习笔记
---

# 1. SpringCloud 简介

微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务。服务之间互相协调、互相配合、为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务间采用轻量级的通讯机制互相协作（通常是基于HTTP协议的RESTful）。每个服务都围绕着其本业务进行构建，并能够被独立的部署到生产环境、类生产环境等。另外，应当尽量避免同一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建。

Spring Cloud=分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶。

# 2. 什么是Spring Cloud

[Spring Cloud](https://projects.spring.io/spring-cloud/)是一个基千Spring Boot实现的微服务架构开发工具。它为微服务架构中涉及的配置管理、服务治理、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。Spring Cloud的诞生并不是为了解决微服务中的某一个问题，而是提供了一套解决微服务架构实施的综合性解决方案。

Spring Cloud是一个由各个独立项目组成的综合项目，每个独立项目有着不同的发布节奏，为了管理每个版本的子项目清单，避免Spring Cloud的版本号与其子项目的版本号相混淆，没有采用版本号的方式，而是通过命名的方式。这些版本的名字采用了伦敦地铁站的名字，根据字母表的顺序来对应版本时间顺序。比如”Angel”是Spring Cloud的第一个发行版名称, “Brixton”是Spring Cloud的第二个发行版名称。当一个版本的Spring Cloud项目的发布内容积累到临界点或者一个严重bug解决可用后，就会发布一个”service releases”版本，简称SRX版本，其中X是一个递增的数字，所以Brixton.SR5就是Brixton的第5个Release版本。

# 3. Spring Cloud 技术栈

- 服务注册和服务发现 Eureka
- 服务负载与调用  Ribbon / Feign
- 服务熔断降级 Hystrix
- 服务网关 Zuul 
- 服务分布式配置 Spring Cloud Config
- 服务开发 Spring Cloud

# 4. SpringCloud与SpringBoot兼容性

方位下面连接获取JSON数据查看兼容版本。

<https://start.spring.io/actuator/info> 

| Release Train | Boot Version |
| ------------- | ------------ |
| Hoxton        | 2.2.x        |
| Greenwich     | 2.1.x        |
| Finchley      | 2.0.x        |
| Edgware       | 1.5.x        |
| Dalston       | 1.5.x        |

# 5. 学习版本

本次学习的版本如下：

- SpringBoot——2.2.2.RELEASE
- SpringCloud——Hoxton.SR1
- SpringCloud alibaba——2.1.0.RELEASE
- java——Java8
- Maven——3.5及以上
- Mysql——5.7及以上

