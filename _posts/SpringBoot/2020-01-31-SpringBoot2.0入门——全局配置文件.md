---
layout:     post
title:      SpringBoot2.0入门——全局配置文件
subtitle:   
date:       2020-01-31
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0入门
---

......

## 1 配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

- application.properties
- application.yml

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们自动配置好；

标记语言：

​	以前的配置文件；大多都使用的是  **xxxx.xml**文件；

​	YAML：**以数据为中心**，比json、xml等更适合做配置文件；

# 2 YAML配置例子

文件名称：application.yml

```yaml
server:
  port: 8082
```

## 2.1 基本语法

- k:(空格)v：标识一对键值对（空格必须有）
- 以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的
- 属性和值也是大小写敏感

## 2.2 值的写法

#### 字面量：普通的值（数字，字符串，布尔）

​	k: v：字面直接来写；

​		字符串默认不用加上单引号或者双引号；

​		""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思

​				name:   "zhangsan \n lisi"：输出；zhangsan 换行  lisi

​		''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

​				name:   ‘zhangsan \n lisi’：输出；zhangsan \n  lisi

#### 对象、Map（属性和值）（键值对）：

​	k: v：在下一行来写对象的属性和值的关系；注意缩进

​		对象还是k: v的方式

```yaml
friends:
		lastName: zhangsan
		age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```

#### 数组（List、Set）：

用- 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

## 2.3 YMAL配置文件值注入

导入配置文件处理器，配置文件进行绑定就会有提示

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

配置文件

```yaml
person:
  lastName: 张三
  age: 27
  boss: false
  birth: 2020/01/31
  maps: {k1: v1,K2: v2}
  lists:
    - 赵四
    - 赵柳
  dog:
    name: 小狗
    age: 2
```

javaBean

```java
package com.chaohappy.springboot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值；
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
@EnableConfigurationProperties(Person.class)
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```

```java
package com.chaohappy.springboot.bean;

public class Dog {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

测试：

```java
package com.chaohappy.springboot;

import com.chaohappy.springboot.bean.Person;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
class SpringBootConfigApplicationTests {

    @Autowired
    Person person;

    @Test
    void contextLoads() {
        System.out.println(person);
    }
}
```

## 2.4 properties文件值注入

```properties
#配置person的值
person.last-name=张三
person.age=27
person.maps.k1=v1
person.lists=a,b,c
person.dog.name=旺旺
person.dog.age=2
```

其他同2.3节

## 2.5 @Value获取值

```
@Component
public class Person {
    @Value("person.last-name")
    private String lastName;
    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private Boolean boss;
...    
}
```

@Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

## 2.6 配置文件注入值数据校验

```
@Component
@ConfigurationProperties(prefix = "person")
@EnableConfigurationProperties(Person.class)
@Validated
public class Person {
    //lastName必须是邮箱格式
    @Email
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;
```

## 2.7  @PropertySource加载指定的配置文件

在resource文件夹下创建person.properties文件

```properties
#配置person的值
person.last-name=张三
person.age=27
person.maps.k1=v1
person.lists=a,b,c
person.dog.name=旺旺
person.dog.age=2
```

```java
@Component
// 加载person.properties配置文件
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
@EnableConfigurationProperties(Person.class)
@Validated
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;
...
```

## 2.8 @ImportResource导入Spring的配置文件

导入Spring的配置文件，让配置文件里面的内容生效

创建bean.xml Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="helloService" class="com.chaohappy.springboot.bean.HelloService"></bean>

</beans>
```

创建HelloService类

```java
package com.chaohappy.springboot.bean;

public class HelloService {
}
```

在主程序类上增加注解@ImportResource(locations = {"classpath:bean.xml"})

```java
@ImportResource(locations = {"classpath:bean.xml"})
@SpringBootApplication
public class SpringBootConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootConfigApplication.class, args);
    }

}
```

测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringBootConfigApplicationTests {

    @Autowired
    ApplicationContext ioc;

    @Test
    public void testHelloService(){
        boolean helloService = ioc.containsBean("helloService");
        System.out.println("容器中有helloService："+helloService);
    }
}
```

测试结果：容器中有helloService：true

## 2.9 SpringBoot 推荐使用全注解的方式给容器中添加组件

创建配置类：MyAppConfig

在配置类上增加注解：@Configuration，作用：指明当前类是配置类；用来替换Spring的配置文件

编写返回service实例对象的方法，并在方法上增加@Bean注解，作用：将方法的返回值添加到容器中；容器中这个自建默认的id就是方法名

```java
/**
 * @Configuration 指明当前类是配置类；用来替换Spring的配置文件
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个自建默认的id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

测试：

配置类@Bean给容器中添加组件了...

容器中有helloService：true