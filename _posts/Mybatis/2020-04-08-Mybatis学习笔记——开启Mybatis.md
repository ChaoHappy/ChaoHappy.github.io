---
layout:     post
title:      Mybatis学习笔记——开启Mybatis
subtitle:   
date:       2020-04-08
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mybatis学习笔记
---

# 1. 准备工作

**创建表**

```sql
CREATE TABLE user (
	id  int(10) NOT NULL AUTO_INCREMENT ,
	name  varchar(50) NULL ,
	pwd  varchar(50) NULL ,
PRIMARY KEY (id)
);
```

**创建工程**

1. 创建一个普通的Maven项目

2. 导入依赖

   ```xml
       <dependencies>
           <!-- mysql驱动 -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.46</version>
           </dependency>
           <!-- mybatis -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.3</version>
           </dependency>
           <!-- junit -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   	<build>
           <resources>
               <resource>
                   <directory>src/main/java</directory>
                   <includes>
                       <include>**/*.xml</include>
                   </includes>
               </resource>
           </resources>
       </build>
   ```

# 2. 编写核心配置文件 

**mybatis-config.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- configuration核心配置文件 -->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3308/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="1111"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 每一个Mapper.xml都需要在mybatis核心配置文件中注测 -->
    <mappers>
        <mapper resource="com/learn/mybatis/dao/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

# 3. 构建SqlSessionFactory

利用mybatis-config.xml完成SqlSessionFactory的构建，并创建SqlSession。采用单例的形式构建SqlSessionFactory。

```java
package com.chp.mybatis.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class SqlSessionFactoryUtil {
    private static SqlSessionFactory sqlSessionFactory = null;
    //类线程锁
    private static final Class CLASS_LOCK = SqlSessionFactoryUtil.class;
    //私有化构造函数
    private SqlSessionFactoryUtil(){}
    //构建SqlSessionFactory
    public static SqlSessionFactory initSqlSessionFactory(){
        String resource = "mybatis-config.xml";
        InputStream in = null;
        try {
            in = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
        synchronized (CLASS_LOCK) {
            if(sqlSessionFactory == null){
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
            }
        }
        return sqlSessionFactory;
    }

    //创建SqlSession
    public static SqlSession openSqlSession(){
        if(sqlSessionFactory == null){
            initSqlSessionFactory();
        }
        return sqlSessionFactory.openSession();
    }
}
```

# 4. POJO

创建一个与库表对应的POJO：

```java
package com.chp.mybatis.pojo;

public class User {
    private Long id;

    private String name;

    private String pwd;
...
}
```

# 5. 接口与映射文件

新建一个UserMapper接口，包含简单的CRUD抽象方法： 

```java
public interface UserMapper {
    User getUser(Long id);
    void createUser(User user);
    void updateUser(@Param("id")Long id,@Param("name") String name);
    void deleteUser(Long id);
}
```

编写UserMapper.xml映射文件，让其自动映射RoleMapper interface：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--和接口路径和名称保持一致，MyBatis会自动帮我们找到这个 Mapper-->
<mapper namespace="com.chp.mybatis.mapper.UserMapper">
    <!-- id与接口方法名一致
        parameterType参数类型与接口方法参数类型一致
        resultType返回值类型与接口方法一致
        'user'为mybatis-config.xml中定义的别名 -->
    <select id="getUser" parameterType="long" resultType="user">
        <![CDATA[
            select * from user where id=#{id}
        ]]>
    </select>

    <insert id="createUser" parameterType="user">
        <![CDATA[
            insert into user (name,pwd)values (#{name},#{pwd})
        ]]>
    </insert>

    <update id="updateUser">
        <![CDATA[
            update user set name =#{name,jdbcType=VARCHAR} where id=#{id,jdbcType=BIGINT}
        ]]>
    </update>

    <delete id="deleteUser" parameterType="long">
            delete from user where id=#{id}
    </delete>
</mapper>
```

# 6. 测试

```java
package com.chp.mybatis.mapper;

import com.chp.mybatis.pojo.User;
import com.chp.mybatis.utils.SqlSessionFactoryUtil;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

public class UserMapperTest {

    @Test
    public void insertUserTest(){
        SqlSession sqlSession = null;
        try {
            sqlSession =SqlSessionFactoryUtil.openSqlSession();
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            User user = new User();
            user.setName("张三");
            user.setPwd("123456");
            userMapper.createUser(user);
            sqlSession.commit();
        }catch (Exception e){
            e.printStackTrace();
            sqlSession.rollback();
        }finally{
            if(sqlSession != null){
                sqlSession.close();
            }
        }
    }

    @Test
    public void getUserTest(){
        SqlSession sqlSession = null;
        try {
            sqlSession =SqlSessionFactoryUtil.openSqlSession();
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            User user = userMapper.getUser(1l);
            System.out.println(user);
            sqlSession.commit();
        }catch (Exception e){
            e.printStackTrace();
            sqlSession.rollback();
        }finally{
            if(sqlSession != null){
                sqlSession.close();
            }
        }
    }

    @Test
    public void updateUserTest(){
        SqlSession sqlSession = null;
        try {
            sqlSession =SqlSessionFactoryUtil.openSqlSession();
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            userMapper.updateUser(1l,"李四");
            sqlSession.commit();
        }catch (Exception e){
            e.printStackTrace();
            sqlSession.rollback();
        }finally{
            if(sqlSession != null){
                sqlSession.close();
            }
        }
    }

    @Test
    public void deleteUserTest(){
        SqlSession sqlSession = null;
        try {
            sqlSession =SqlSessionFactoryUtil.openSqlSession();
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            userMapper.deleteUser(1l);
            sqlSession.commit();
        }catch (Exception e){
            e.printStackTrace();
            sqlSession.rollback();
        }finally{
            if(sqlSession != null){
                sqlSession.close();
            }
        }
    }
}
```

# 7. 采坑解决方案

<https://blog.csdn.net/u010648555/article/details/70880425> 

<https://www.jianshu.com/p/fd68ca404d94> 

 

------

[代码地址](https://github.com/ChaoHappy/MybatisAll/tree/master/01.Start-Mybatis )

