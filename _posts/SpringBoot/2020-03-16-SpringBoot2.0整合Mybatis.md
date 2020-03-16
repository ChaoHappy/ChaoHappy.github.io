---
layout:     post
title:      SpringBoot整合Mybatis
subtitle:   
date:       2020-03-16
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:

- SpringBoot
- Mybatis
---

# 1. 开发前准备
## 1.1 环境参数
- 开发工具 eclipse、Maven、JDK8、Tomcat
- 技术SpringBoot+Mybatis
- 数据库Mysql
- SpringBoot 2.0

## 1.2 配置pom文件
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 1.3 配置数据源——application.yml
```
#mysql
spring:
  datasource:
    url: jdbc:mysql://localhost:3308/eboot-admin?useunicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: 1111
    driver-class-name: com.mysql.jdbc.Driver
```

# 2. 全注解的方式
在mapper接口增加@Mapper注解或在配置类上增加@MapperScan("com.**.mapper")

```
@Mapper
public interface UserMapper {
	
	/**
	 * @return
	 */
	@Select("select * from sys_user where id=#{user_id}")
	SysUser queryUserById(@Param(value = "user_id") int id);
	
	
	@Select("select * from sys_user where login_name=#{login_name}")
	SysUser queryUserByUsername(@Param(value = "login_name") String loginName);
	
	/**
	 */
	@Select("select * from sys_user")
	List<SysUser> queryUsers();
}
```

# 3. XML的方式
## 3.1 dao层
UserMapper接口
```
@Mapper
public interface UserMapper {
	
	/**
	 * @return
	 */
	SysUser queryUserById(@Param(value = "user_id") int id);
	
```
UserMapper.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- UserMapper接口全路径 -->
<mapper namespace="com.chaohappy.system.mapper.UserMapper">

	<!-- id 为接口方法名 -->
    <select id="queryUserById" parameterType="String" resultType="com.chaohappy.system.domain.SysUser">
        select * from sys_user where id=#{user_id}
    </select>
</mapper>
```

## 3.2 配置文件——application.yml
```
# Mybatis
mybatis:
  type-aliases-package: com.chaohappy.system.mapper
  mapper-locations:
  - mapper/*.xml
```