---
layout:     post
title:      eclipse创建SpringBoot项目报错
subtitle:   报错Unknown line 1 Maven Configuration Problem
date:       2020-03-14
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:

- SpringBoot
- eclipse
---



问题：通过eclipse STS插件创建SpringBoot项目 pom文件第一行报错
Unknown	pom.xml		line 1	Maven Configuration Problem

原因：springboot版本太高了，我的eclipse创建后的项目默认是2.1.13.RELEASE版本。

解决方法：将2.1.13.RELEASE版本这个版本改成2.1.4.RELEASE版本