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

# 2. Spring Cloud 技术栈

- 服务注册和服务发现 Eureka
- 服务负载与调用  Ribbon / Feign
- 服务熔断降级 Hystrix
- 服务网关 Zuul 
- 服务分布式配置 Spring Cloud Config
- 服务开发 Spring Cloud





