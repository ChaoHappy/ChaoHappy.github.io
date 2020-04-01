---
layout:     post
title:      SpringBoot2.0教程——开启SpringBoot
subtitle:   
date:       2020-04-01
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
---

Spring Boot是在Spring框架上创建的一个全新的框架，其设计目的是简化Spring应用的搭建和开发过程。开启Spring Boot有许多种方法可供选择，这里仅介绍使用eclipse的STS插件来构建一个简单的Spring Boot项目。 

# 1. 创建maven工程

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\1.png)

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\2.png)

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\3.png)

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\4.png)

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\5.png)

**注意：这个时候项目会报错，原因和解决方案参照这个链接**    [解决方法](https://chaohappy.github.io/2020/03/14/eclipse%E5%88%9B%E5%BB%BASpringBoot%E9%A1%B9%E7%9B%AE%E6%8A%A5%E9%94%99Unknown-line-1-Maven-Configuration-Problem/)

# 2. 简单演示

项目根目录下生成了一个artifactId+Application命名规则的入口类，下面创建控制器类，编写代码：

```java
package com.chp.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class TestController {

    @ResponseBody
	@GetMapping("/hello")
	public String helloWorld() {
		return "HelloWorld";
	}
}
```

然后右键点击项目或Application的主函数，选择run as → Java Application： 

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\6.png)

访问[http://localhost:8080](http://localhost:8080/)，页面显示如下： 

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\7.png)

# 3. 打包发布

在eclipse中右击项目，选择run as → Maven build…，如下图所示：

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\8.png)

在Goals中输入`clean package`命令，然后点击下方的run就将项目打包成jar包（初次打包会自动下载一些依赖）。打包完毕后可看到项目目录target文件夹下生成了一个jar文件： 

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\9.png)

生成jar包后，通过cmd  cd到target目录下，执行以下命令`java -jar 01.Start-Spring-Boot-0.0.1-SNAPSHOT.jar ` ： 

![](https://chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\开启SpringBoot\10.png)

访问[http://localhost:8080](http://localhost:8080/)，页面显示同上。

# 4. 聊聊pom文件

打开pom.xml可看到配置如下： 

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.chp</groupId>
	<artifactId>01.Start-Spring-Boot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>01.Start-Spring-Boot</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

## spring-boot-starter-parent

spring-boot-starter-parent指定了当前项目为一个Spring Boot项目，它提供了诸多的默认Maven依赖 。我们进入pom文件进入parent中，再次点进去就会发现,这个dependencies中其实就是一个依赖管理库 

```java
<properties>
...
<spring.version>5.1.6.RELEASE</spring.version>
<spring-amqp.version>2.1.5.RELEASE</spring-amqp.version>
<spring-batch.version>4.1.2.RELEASE</spring-batch.version>
<spring-cloud-connectors.version>2.0.5.RELEASE</spring-cloud-connectors.version>
<spring-data-releasetrain.version>Lovelace-SR6</spring-data-releasetrain.version>
<spring-framework.version>${spring.version}</spring-framework.version>
<spring-hateoas.version>0.25.1.RELEASE</spring-hateoas.version>
<spring-integration.version>5.1.4.RELEASE</spring-integration.version>
<spring-kafka.version>2.2.5.RELEASE</spring-kafka.version>
<spring-ldap.version>2.3.2.RELEASE</spring-ldap.version>
<spring-plugin.version>1.2.0.RELEASE</spring-plugin.version>
<spring-restdocs.version>2.0.3.RELEASE</spring-restdocs.version>
<spring-retry.version>1.2.4.RELEASE</spring-retry.version>
<spring-security.version>5.1.5.RELEASE</spring-security.version>
...
</properties>
```

需要说明的是，并非所有在`<properties>`标签中配置了版本号的依赖都有被启用，其启用与否取决于您是否配置了相应的starter。比如tomcat这个依赖就是spring-boot-starter-web的传递性依赖（下面将会描述到）。

当然，我们可以手动改变这些依赖的版本。比如我想把thymeleaf的版本改为3.0.0.RELEASE，我们可以在pom.xml中进行如下配置：

```
<properties>
   <thymeleaf.version>3.0.0.RELEASE</thymeleaf.version>
</properties>
```

## spring-boot-starter-web

Spring Boot提供了许多开箱即用的依赖模块，这些模块都是以spring-boot-starter-XX命名的。比如要开启Spring Boot的web功能，只需要在pom.xml中配置spring-boot-starter-web即可： 

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

因为其依赖于spring-boot-starter-parent，所以这里可以不用配置version。保存后Maven会自动帮我们下载spring-boot-starter-web模块所包含的jar文件。如果需要具体查看spring-boot-starter-web包含了哪些依赖，我们可以右键项目选择run as → Maven Build…，在Goals中输入命令`dependency:tree`，然后点击run即可在eclipse控制台查看到如下信息： 

```
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.1.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.1.4.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.11.2:compile
[INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.11.2:compile
[INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.26:compile
[INFO] |  |  +- javax.annotation:javax.annotation-api:jar:1.3.2:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.23:runtime
[INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.8:compile
[INFO] |  |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.8:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.9.8:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.8:compile
[INFO] |  |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.9.8:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-tomcat:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.17:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:9.0.17:compile
[INFO] |  |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:9.0.17:compile
[INFO] |  +- org.hibernate.validator:hibernate-validator:jar:6.0.16.Final:compile
[INFO] |  |  +- javax.validation:validation-api:jar:2.0.1.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.3.2.Final:compile
[INFO] |  |  \- com.fasterxml:classmate:jar:1.4.0:compile
[INFO] |  +- org.springframework:spring-web:jar:5.1.6.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-beans:jar:5.1.6.RELEASE:compile
[INFO] |  \- org.springframework:spring-webmvc:jar:5.1.6.RELEASE:compile
[INFO] |     +- org.springframework:spring-aop:jar:5.1.6.RELEASE:compile
[INFO] |     +- org.springframework:spring-context:jar:5.1.6.RELEASE:compile
[INFO] |     \- org.springframework:spring-expression:jar:5.1.6.RELEASE:compile
[INFO] \- org.springframework.boot:spring-boot-starter-test:jar:2.1.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test:jar:2.1.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test-autoconfigure:jar:2.1.4.RELEASE:test
[INFO]    +- com.jayway.jsonpath:json-path:jar:2.4.0:test
[INFO]    |  +- net.minidev:json-smart:jar:2.3:test
[INFO]    |  |  \- net.minidev:accessors-smart:jar:1.2:test
[INFO]    |  |     \- org.ow2.asm:asm:jar:5.0.4:test
[INFO]    |  \- org.slf4j:slf4j-api:jar:1.7.26:compile
[INFO]    +- junit:junit:jar:4.12:test
[INFO]    +- org.assertj:assertj-core:jar:3.11.1:test
[INFO]    +- org.mockito:mockito-core:jar:2.23.4:test
[INFO]    |  +- net.bytebuddy:byte-buddy:jar:1.9.12:test
[INFO]    |  +- net.bytebuddy:byte-buddy-agent:jar:1.9.12:test
[INFO]    |  \- org.objenesis:objenesis:jar:2.6:test
[INFO]    +- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO]    +- org.hamcrest:hamcrest-library:jar:1.3:test
[INFO]    +- org.skyscreamer:jsonassert:jar:1.5.0:test
[INFO]    |  \- com.vaadin.external.google:android-json:jar:0.0.20131108.vaadin1:test
[INFO]    +- org.springframework:spring-core:jar:5.1.6.RELEASE:compile
[INFO]    |  \- org.springframework:spring-jcl:jar:5.1.6.RELEASE:compile
[INFO]    +- org.springframework:spring-test:jar:5.1.6.RELEASE:test
[INFO]    \- org.xmlunit:xmlunit-core:jar:2.6.2:test
[INFO] ------------------------------------------------------------------------
```

上述这些依赖都是隐式依赖于spring-boot-starter-web，我们也可以手动排除一些我们不需要的依赖。

比如spring-boot-starter-web默认集成了tomcat，假如我们想把它换为jetty，可以在pom.xml中spring-boot-starter-web下排除tomcat依赖，然后手动引入jetty依赖：

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

tips：依赖的坐标可以到上述的spring-boot-dependencies-1.5.9.RELEASE.pom文件里查找。再次运行`dependency:tree`： 

```
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.1.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.1.4.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.11.2:compile
[INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.11.2:compile
[INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.26:compile
[INFO] |  |  +- javax.annotation:javax.annotation-api:jar:1.3.2:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.23:runtime
[INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:2.1.4.RELEASE:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.8:compile
[INFO] |  |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.8:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.9.8:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.8:compile
[INFO] |  |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.9.8:compile
[INFO] |  +- org.hibernate.validator:hibernate-validator:jar:6.0.16.Final:compile
[INFO] |  |  +- javax.validation:validation-api:jar:2.0.1.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.3.2.Final:compile
[INFO] |  |  \- com.fasterxml:classmate:jar:1.4.0:compile
[INFO] |  +- org.springframework:spring-web:jar:5.1.6.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-beans:jar:5.1.6.RELEASE:compile
[INFO] |  \- org.springframework:spring-webmvc:jar:5.1.6.RELEASE:compile
[INFO] |     +- org.springframework:spring-aop:jar:5.1.6.RELEASE:compile
[INFO] |     +- org.springframework:spring-context:jar:5.1.6.RELEASE:compile
[INFO] |     \- org.springframework:spring-expression:jar:5.1.6.RELEASE:compile
[INFO] +- org.springframework.boot:spring-boot-starter-jetty:jar:2.1.4.RELEASE:compile
[INFO] |  +- org.eclipse.jetty:jetty-servlets:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty:jetty-continuation:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty:jetty-http:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty:jetty-util:jar:9.4.15.v20190215:compile
[INFO] |  |  \- org.eclipse.jetty:jetty-io:jar:9.4.15.v20190215:compile
[INFO] |  +- org.eclipse.jetty:jetty-webapp:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty:jetty-xml:jar:9.4.15.v20190215:compile
[INFO] |  |  \- org.eclipse.jetty:jetty-servlet:jar:9.4.15.v20190215:compile
[INFO] |  |     \- org.eclipse.jetty:jetty-security:jar:9.4.15.v20190215:compile
[INFO] |  |        \- org.eclipse.jetty:jetty-server:jar:9.4.15.v20190215:compile
[INFO] |  +- org.eclipse.jetty.websocket:websocket-server:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty.websocket:websocket-common:jar:9.4.15.v20190215:compile
[INFO] |  |  |  \- org.eclipse.jetty.websocket:websocket-api:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty.websocket:websocket-client:jar:9.4.15.v20190215:compile
[INFO] |  |  |  \- org.eclipse.jetty:jetty-client:jar:9.4.15.v20190215:compile
[INFO] |  |  \- org.eclipse.jetty.websocket:websocket-servlet:jar:9.4.15.v20190215:compile
[INFO] |  |     \- javax.servlet:javax.servlet-api:jar:4.0.1:compile
[INFO] |  +- org.eclipse.jetty.websocket:javax-websocket-server-impl:jar:9.4.15.v20190215:compile
[INFO] |  |  +- org.eclipse.jetty:jetty-annotations:jar:9.4.15.v20190215:compile
[INFO] |  |  |  +- org.eclipse.jetty:jetty-plus:jar:9.4.15.v20190215:compile
[INFO] |  |  |  +- org.ow2.asm:asm:jar:7.0:compile
[INFO] |  |  |  \- org.ow2.asm:asm-commons:jar:7.0:compile
[INFO] |  |  |     +- org.ow2.asm:asm-tree:jar:7.0:compile
[INFO] |  |  |     \- org.ow2.asm:asm-analysis:jar:7.0:compile
[INFO] |  |  +- org.eclipse.jetty.websocket:javax-websocket-client-impl:jar:9.4.15.v20190215:compile
[INFO] |  |  \- javax.websocket:javax.websocket-api:jar:1.1:compile
[INFO] |  \- org.mortbay.jasper:apache-el:jar:8.5.35.1:compile
[INFO] \- org.springframework.boot:spring-boot-starter-test:jar:2.1.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test:jar:2.1.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test-autoconfigure:jar:2.1.4.RELEASE:test
[INFO]    +- com.jayway.jsonpath:json-path:jar:2.4.0:test
[INFO]    |  +- net.minidev:json-smart:jar:2.3:test
[INFO]    |  |  \- net.minidev:accessors-smart:jar:1.2:test
[INFO]    |  \- org.slf4j:slf4j-api:jar:1.7.26:compile
[INFO]    +- junit:junit:jar:4.12:test
[INFO]    +- org.assertj:assertj-core:jar:3.11.1:test
[INFO]    +- org.mockito:mockito-core:jar:2.23.4:test
[INFO]    |  +- net.bytebuddy:byte-buddy:jar:1.9.12:test
[INFO]    |  +- net.bytebuddy:byte-buddy-agent:jar:1.9.12:test
[INFO]    |  \- org.objenesis:objenesis:jar:2.6:test
[INFO]    +- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO]    +- org.hamcrest:hamcrest-library:jar:1.3:test
[INFO]    +- org.skyscreamer:jsonassert:jar:1.5.0:test
[INFO]    |  \- com.vaadin.external.google:android-json:jar:0.0.20131108.vaadin1:test
[INFO]    +- org.springframework:spring-core:jar:5.1.6.RELEASE:compile
[INFO]    |  \- org.springframework:spring-jcl:jar:5.1.6.RELEASE:compile
[INFO]    +- org.springframework:spring-test:jar:5.1.6.RELEASE:test
[INFO]    \- org.xmlunit:xmlunit-core:jar:2.6.2:test
[INFO] ------------------------------------------------------------------------
```

可看到tomcat已被替换为了jetty。 

## spring-boot-maven-plugin

spring-boot-maven-plugin为Spring Boot Maven插件，提供了：

1. 把项目打包成一个可执行的超级JAR（uber-JAR）,包括把应用程序的所有依赖打入JAR文件内，并为JAR添加一个描述文件，其中的内容能让你用`java -jar`来运行应用程序。
2. 搜索`public static void main()`方法来标记为可运行类。