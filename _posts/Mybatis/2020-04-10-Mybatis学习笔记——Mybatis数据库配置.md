---
layout:     post
title:      Mybatis学习笔记——Mybatis数据库配置
subtitle:   
date:       2020-04-10
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mybatis
    - Mybatis学习笔记
---

# 1. xml文件配置

直接在mybatis-config.xml文件中配置： 

```xml
<dataSource type="POOLED">
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3308/mybatis_learn?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
    <property name="username" value="root"/>
    <property name="password" value="1111"/>
</dataSource>
```

# 2. properties配置文件

为了方便日后维护修改，我们用properties配置文件来配置数据库属性： 

db.properties： 

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3308/mybatis_learn?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8
username=root
password=1111
```

在mybatis-config.xml文件中引入： 

```xml
......
<dataSource type="POOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="{password}}"/>
</dataSource>
......
```

# 3. 参数传递

假如要对db.properties文件中的用户名和密码进行加密，那我们则需要在生成SqlSessionFactory的时候对用户名和密码解密（假设解密方法为`decode()`）： 

```java
public class SqlSessionFactoryUtil {
    InputStream cfgStream = null;
    Reader cfgReader = null;
    InputStream proStream = null;
    Reader proReader = null;
    Properties properties = null;
    private static SqlSessionFactory sqlSessionFactory = null;
    //类线程锁
    private static final Class CLASS_LOCK = SqlSessionFactoryUtil.class;
    //私有化构造函数
    private SqlSessionFactoryUtil(){}
    //构建SqlSessionFactory
    public static SqlSessionFactory initSqlSessionFactory(){
        try{
            //读入配置文件流
            cfgStream = Resources.getResourceAsStream("mybatis-config.xml");
            cfgReader = new InputStreamReader(cfgStream);
            //读入属性文件
            proStream = Resources.getResourceAsStream(db.properties);
            proReader = new InputStreamReader(proStream);
            
            properties = new Properties();
            properties.load(proReader);
            properties.setProperty("username",
                decode(properties.getProperty("username")));
            properties.setProperty("password",
                decode(properties.getProperty("password")));
        }catch(IOException e){
            e.printStackTrace();
        }
        synchronized (CLASS_LOCK) {
            if(sqlSessionFactory == null){
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(cfgStream);
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

# 4. environments配置环境

```xml
    <!--数据源配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 采用JDBC事物管理器 -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="{password}}"/>
            </dataSource>
        </environment>
    </environments>
```

`default`属性表明默认选用哪个数据库。

`id`属性为一个数据库配置的标识，可以同时配置多个数据库。

dataSource的`type`属性可选非连接池UNPOOLED，连接池POOLED和JNDI。

[^参考文档]: <https://mrbird.cc/MyBatis%E9%85%8D%E7%BD%AE%E6%95%B0%E6%8D%AE%E5%BA%93.html> 



[源码地址](https://github.com/ChaoHappy/MybatisAll/tree/master/02.Mybatis-DB-Config)

















