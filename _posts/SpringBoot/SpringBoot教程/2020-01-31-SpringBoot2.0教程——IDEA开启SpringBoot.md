---
layout:     post
title:      SpringBoot2.0教程——IDEA开启SpringBoot
subtitle:   
date:       2020-01-30
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
---

# 1 创建maven工程（jar）

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/0.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/1.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/2.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/3.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/4.png)



![1](D:\chaohappy\github\chaohappy.github.io\images\SpringBoot-学习\SpringBoot-HelloWorld\5.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/6.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/7.png)



![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/8.png)

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/9.png)

# 2 导入Spring Boot相关的依赖

pom.xml文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

# 3 编写主程序—启动SpringBoot 应用程序

```java
/**
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class);
    }
}
```

# 4 编写相关的Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World";
    }
}
```

# 5 测试

## 启动应用程序

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/10.png)

## 访问url <http://localhost:8080/hello> 

![1](https://chaohappy.github.io/images/SpringBoot-学习/SpringBoot-HelloWorld/11.png)

