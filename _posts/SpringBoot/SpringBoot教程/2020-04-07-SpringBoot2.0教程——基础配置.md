---
layout:     post
title:      SpringBoot2.0教程——基础配置
subtitle:   
date:       2020-04-07
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
---

# 1. 全局配置文件

在src/main/resources目录下，Spring Boot提供了一个全局配置文件，可对一些默认配置的配置值进行修改。

配置文件名是固定的：

- application.properties
- application.yml

下面主要介绍application.yml配置文件。

## 1.1. 自定义属性值

Spring Boot允许我们在application.yml下自定义一些属性，在开始之前添加配置文件处理器插件，添加后编写配置文件就有提示了。在pom.xml文件添加如下依赖：

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

在application.yml下自定义一些属性，比如：

```yaml
person:
  name: 张三
  age: 18
```

定义一个Person1 Bean，通过@Value("${属性名}")来加载配置文件中的属性值：

```java
@Component
public class Person1 {
	
	@Value("${person.name}")
	private String name;
	
	@Value("${person.age}")
	private Integer age;
...
}
```

编写IndexController，注入该Bean：

```java
@RestController
public class IndexController {

    @Autowired
    private Person1 person1;

    @GetMapping("/person1")
    public Person1 person1(){
        return person1;
    }
}
```

启动项目，访问http://localhost:8080/person1，页面显示如下：

![](D:\chaohappy\github\chaohappy.github.io\images\SpringBoot-学习\SpringBoot教程\全局配置文件\1.png)

在属性非常多的情况下，也可以定义一个和配置文件对应的Bean：

```java
@ConfigurationProperties(prefix = "person")
public class Person2 {

    private String name;

    private Integer age;
...
}
```

通过注解@ConfigurationProperties(prefix="person")指明了属性的通用前缀，通用前缀加属性名和配置文件的属性名一一对应。

除此之外还需在Spring Boot入口类加上注解@EnableConfigurationProperties({Person2.class})来启用该配置：

```java
@EnableConfigurationProperties({Person2.class})
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication app = new SpringApplication(Application.class);
//        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
	}
}
```

编写IndexController，注入该Bean：

```java
@GetMapping("/person2")
    public Person2 getPerson2() {
    return person2;
}
```

启动项目，访问http://localhost:8080/person2，页面显示如上图所示。

## 1.2 属性间的引用

在application.yml配置文件中，各个属性可以相互引用，如下： 

```yaml
person:
  name: 张三
  age: 18

test:
  name: ${person.name}
```

# 2. 自定义配置文件

除了可以在application.yml里配置属性，我们还可以自定义一个配置文件。在src/main/resources目录下新建一个person3.yml:

```yaml
person3:
  name: 张三
  age: 181
```

因为这个文档是以YAML文件为例，注解@PropertySource默认不支持YAML文件，需要重写如下接口：

```java
public class YamlPropertySourceFactory implements PropertySourceFactory {
    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
        factory.setResources(resource.getResource());
        factory.afterPropertiesSet();
        Properties ymlProperties = factory.getObject();
        String propertyName = name != null ? name : resource.getResource().getFilename();
        return new PropertiesPropertySource(propertyName, ymlProperties);
    }
}
```

定义一个对应该配置文件的Bean：

```yaml
@ConfigurationProperties(prefix = "person3")
@PropertySource(value = "classpath:person3.yml",factory = YamlPropertySourceFactory.class)
@Component
public class Person3 {

    private String name;

    private Integer age;
...
}
```

注解@PropertySource("classpath:person3.yml")指明了使用哪个配置文件。要使用该配置Bean，同样也需要在入口类里使用注解@EnableConfigurationProperties({Person3.class})来启用该配置。

# 3. 通过命令行设置属性值

在运行Spring Boot jar文件时，可以使用命令java -jar xxx.jar --server.port=8081来改变端口的值。这条命令等价于我们手动到application.properties中修改（如果没有这条属性的话就添加）server.port属性的值为8081。

如果不想项目的配置被命令行修改，可以在入口文件的main方法中进行如下设置：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(Application.class);
    app.setAddCommandLineProperties(false);
    app.run(args);
}
```

# 4. 使用XML配置

虽然Spring Boot并不推荐我们继续使用xml配置，但如果出现不得不使用xml配置的情况，Spring Boot允许我们在入口类里通过注解@ImportResource({"classpath:some-application.xml"})来引入xml配置文件。

# 5. Profile配置

Profile用来针对不同的环境下使用不同的配置文件，多环境配置文件必须以application-{profile}.yml的格式命，其中`{profile}`为环境标识。比如定义两个配置文件：

- application-dev.yml：开发环境

```
server.port=8080
```

application-prod.yml：生产环境 

```
server.port=8081
```

至于哪个具体的配置文件会被加载，需要在application.yml文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=dev`就会加载application-dev.yml配置文件内容。可以在运行jar文件的时候使用命令`java -jar xxx.jar --spring.profiles.active={profile}`切换不同的环境配置。











