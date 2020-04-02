---
layout:     post
title:      SpringBoot2.0教程——IDEA快速创建项目
subtitle:   
date:       2020-01-32
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
    - IDEA
---

## 1  使用Spring Initializer快速创建SpringBoot项目

![1](https://chaohappy.github.io/images/SpringBoot-学习/IDEA快速创建项目/1.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/IDEA快速创建项目/2.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/IDEA快速创建项目/3.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/IDEA快速创建项目/4.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/IDEA快速创建项目/5.png)

## 2 创建HelloWorldController

```java
package com.chaohappy.springboot.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@ResponseBody
@Controller
public class HelloController {

    @RequestMapping("/hello")
    public String hello(){
        return "HelloWorld!";
    }
}

```



## 3 修改Tomcat端口号

修改文件application.properties，增加 server.port=8081

## 4 测试

执行主程序

访问url：<http://localhost:8081/hello> 

## 5 总结

默认生成的Spring Boot项目；

- 主程序已经生成好了，我们只需要我们自己的逻辑
- resources文件夹中目录结构
  - static：保存所有的静态资源； js css  images；
  - templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
  - application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；



