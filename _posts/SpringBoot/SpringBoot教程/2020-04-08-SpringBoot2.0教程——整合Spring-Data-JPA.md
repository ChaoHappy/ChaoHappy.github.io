---
layout:     post
title:      SpringBoot2.0教程（四）
subtitle:   整合Spring-Data-JPA
date:       2020-04-08
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
---

# 1.POM文件 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

# 2. 在**application.yml** 中配置数据库信息和JPA



```yaml
server:
  port: 8888
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3308/mybatis_learn?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8
    data-username: root
    data-password: 1111
  jpa:
    hibernate:
      ddl-auto: update

```

# 3. 创建实体类

```java
@Entity
@Table(name = "t_user")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column
    private String name;
    
    //省略get/set方法
}
```

1. @Entity 是一个必选的注解，声明这个类对应了一个数据库表。
2. @Table(name = "t_user") 是一个可选的注解。声明了数据库实体对应的表信息。包括表名称、索引信息等。这里声明这个实体类对应的表名是 t_user。如果没有指定，则表名和实体的名称保持一致。
3. @Id 注解声明了实体唯一标识对应的属性。
4. @GeneratedValue注解存在的意义主要就是为一个实体生成一个唯一标识的主键(JPA要求每一个实体Entity,必须有且只有一个主键),@GeneratedValue提供了主键的生成策略。
5. @Column用来声明实体属性的表字段的定义。默认的实体每个属性都对应了表的一个字段。字段的名称默认和属性名称保持一致（并不一定相等）。字段的类型根据实体属性类型自动推断。

# 4. 数据库

```sql
CREATE TABLE `NewTable` (
`id`  int(10) NOT NULL AUTO_INCREMENT ,
`name`  varchar(50) NULL ,
PRIMARY KEY (`id`)
);
```

# 5.创建数据库访问Dao层 

```java
@Repository
public interface UserDao extends JpaRepository<User,Long> {

   @Override
   User  save(User s);
}
```

复写父类中的方法就可以实现增删改查了，自己写个例子试试就知道了，不详细总结了。

父类中还有很多可以使用的内置增删改查的方法 。

# 6. 测试

```java
@Test
public void saveUserTest(){
    User user = new User();
    user.setName("张四");
    userDao.save(user);
    List<User> users = userDao.findAll();
    System.out.println("users："+users);
}
```

