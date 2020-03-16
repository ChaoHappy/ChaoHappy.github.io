---
layout:     post
title:      SpringBoot整合mybatis查询数据返回实体类时，实体类个别字段值为null的问题
subtitle:   
date:       2020-03-16
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:

- SpringBoot
- Mybatis
---

我遇到的问题的原因是没有开启mybatis的字段命名驼峰转换，导致某些表的字段名找不到实体类的对应的属性。在 application.yml添加

mybatis:
  configuration:
    map-underscore-to-camel-case: true