---
layout:     post
title:      Mybatis学习笔记——TypeHandler
subtitle:   
date:       2020-04-10
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mybatis
    - Mybatis学习笔记
---

MyBatis在设置参数或者从结果集中获取参数的时候，都会用到注册了的typeHandler进行处理。typeHandler的作用为将参数从javaType转为jdbcType，或者从数据库取出结果时把jdbcType转为javaType。 

# 1. 内置TypeHandler

| 类型处理器              | Java类型                  | JDBC类型                                       |
| ----------------------- | ------------------------- | ---------------------------------------------- |
| BooleanTypeHandler      | java.lang.Boolean,boolean | 数据库兼容的BOOLEAN                            |
| ByteTypeHandler         | java.lang.Byte,byte       | 数据库兼容的NUMERIC或BYTE                      |
| ShortTypeHandler        | java.lang.Short,short     | 数据库兼容的NUMERIC或SHORT INTEGER             |
| IntegerTypeHandler      | java.lang.Integer,int     | 数据库兼容的NUMERIC或INTEGER                   |
| LongTypeHandler         | java.lang.Long,long       | 数据库兼容的NUMERIC或LONG INTEGER              |
| FloatTypeHandler        | java.lang.Float,float     | 数据库兼容的NUMERIC或FLOAT                     |
| DoubleTypeHandler       | java.lang.Double,double   | 数据库兼容的NUMERIC或DOUBLE                    |
| BigDecimalTypeHandler   | java.math.BigDecimal      | 数据库兼容的NUMERIC或DECIMAL                   |
| StringTypeHandler       | java.lang.Stirng          | CHAR,VARCHAR                                   |
| ClobypeHandler          | java.lang.String          | CLOB,LONGVARCHAR                               |
| NStringTypeHanler       | java.lang.String          | NVARCHAR,NCHAR                                 |
| NClobTypeHandler        | java.lang.String          | NNCLOB                                         |
| ByteArrayTypeHandler    | byte[]                    | 数据库兼容的字节流类型                         |
| BlobTypeHandler         | byte[]                    | BLOB,LONGVARBINARY                             |
| DateTypeHandler         | java.util.Date            | TIMESTAMP                                      |
| DateOnlyTypeHandler     | java.util.Date            | DATE                                           |
| TimeOnlyTypeHandler     | java.util.Date            | TIME                                           |
| SqlTimestampTypeHandler | java.sql.Timestamp        | TIMESTAMP                                      |
| SqlDateTypeHandler      | java.sql.Date             | DATE                                           |
| SqlTimeTypeHandler      | java.sql.Time             | TIME                                           |
| ObjectTypeHandler       | Any                       | OTHER或未指定类型                              |
| EnumTypeHandler         | Enumeration Type          | VARCHAR或任意兼容的字符串类型，存 储枚举的名称 |
| EnumOrdinalTypeHandler  | Enumeration Type          | 任何兼容的NUMERIC或DOUBLE类型， 存储枚举的索引 |

为了演示内置的TypeHandler，新建一张表： 

```sql
CREATE TABLE t_role (
`id`  int(20) NOT NULL AUTO_INCREMENT COMMENT 'id' ,
`role_name`  varchar(60) NULL COMMENT '角色名称' ,
`remark`  varchar(1024) NULL COMMENT '备注' ,
`is_admin`  varchar(20) NULL COMMENT '是否为管理员' ,
PRIMARY KEY (`id`)
);
```

数据库对应的实体类Role

```java
public class Role {

    private Long id;

    private String roleName;

    private String remark;

    private Boolean isAdmin;
......
}
```

接口RoleMapper中定义一个createRole()抽象方法： 

```java
public interface RoleMapper {
    int createRole(Role role);
}
```

映射文件： 

```xml
<!--和接口路径和名称保持一致，MyBatis会自动帮我们找到这个 Mapper-->
<mapper namespace="com.chp.mybatis.mapper.RoleMapper">
    <insert id="createRole" parameterType="role" >
        <![CDATA[
            insert into t_role (role_name,remark,is_admin) values (#{roleName},#{remark},#{isAdmin})
        ]]>
    </insert>
</mapper>
```

查询数据库：

```sql
mysql> select * from t_role;
+----+-----------+--------+----------+
| id | role_name | remark | is_admin |
+----+-----------+--------+----------+
|  1 | 收费员    | NULL   | 0        |
+----+-----------+--------+----------+
```

从结果中可以看出，默认的BooleantypeHandler将false转换为了0。

如果想把true转换为Y,false转换为N,我们可以自定义BooleantypeHandler。

# 2. 自定义TypeHandler

自定义typeHandler可以通过继承BasetypeHandler或者实现TypeHandler接口来实现，现自定义一个BooleanTypeHandler： 

```java
package com.chp.mybatis.typehandler;

import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class BooleanTypeHandler  implements TypeHandler<Boolean> {
    public void setParameter(PreparedStatement preparedStatement, int i, Boolean aBoolean, JdbcType jdbcType) throws SQLException {
        Boolean flag = (Boolean) aBoolean;
        String value = flag == true ? "Y" : "N";
        preparedStatement.setString(i, value);
    }

    public Boolean getResult(ResultSet resultSet, String s) throws SQLException {
        String str = resultSet.getString(s);
        Boolean flag = Boolean.FALSE;
        if(str.equalsIgnoreCase("Y")){
            flag = Boolean.TRUE;
        }
        return flag;
    }

    public Boolean getResult(ResultSet resultSet, int i) throws SQLException {
        String str = resultSet.getString(i);
        Boolean flag = Boolean.FALSE;
        if(str.equalsIgnoreCase("Y")){
            flag = Boolean.TRUE;
        }
        return flag;
    }

    public Boolean getResult(CallableStatement callableStatement, int i) throws SQLException {
        String str = callableStatement.getString(i);
        Boolean flag = Boolean.FALSE;
        if(str.equalsIgnoreCase("Y")){
            flag = Boolean.TRUE;
        }
        return flag;
    }
}
```

在mybatis-config.xml文件中配置该TypeHandler： 

```xml
<typeHandlers>
    <typeHandler javaType="Boolean" jdbcType="VARCHAR" handler="com.chp.mybatis.typehandler.BooleanTypeHandler" />
</typeHandlers>
```

然后在映射文件中对需要转换的字段标注javaType和jdbcType。

指明javaType和jdbcType，与注册中的一致即可找到相对应的typeHandler： 

```xml
    <insert id="createRole" parameterType="role" >
        <![CDATA[
            insert into t_role (role_name,remark,is_admin) values (#{roleName},#{remark},
            #{isAdmin,javaType=Boolean,jdbcType=VARCHAR})
        ]]>
    </insert>
```

或者无需在mybatis-config.xml中注册直接在映射文件中指明typeHandler的路径即可。 

```xml
    <insert id="createRole" parameterType="role" >
        <![CDATA[
            insert into t_role (role_name,remark,is_admin) values (#{roleName},#{remark},
            #{isAdmin,typeHandler=com.chp.mybatis.typehandler.BooleanTypeHandler})
        ]]>
    </insert>
```

测试略。

查询数据库：

```sql
mysql> select * from t_role;
+----+-----------+--------+----------+
| id | role_name | remark | is_admin |
+----+-----------+--------+----------+
|  1 | 收费员    | NULL   | 0        |
|  2 | 收费员1   | NULL   | 0        |
|  3 | 收费员1   | NULL   | N        |
|  4 | 收费员1   | NULL   | Y        |
+----+-----------+--------+----------+
```

# 3. 枚举类型typeHandler

MyBatis自带两种枚举类型处理器： 

1. org.apache.ibatis.type.EnumOrdinalTypeHandler
2. org.apache.ibatis.type.EnumTypeHandler

EnumOrdinalTypeHandler使用整数下标传递，EnumTypeHandler使用枚举字符串传递。 

创建Sex枚举：

```java
public enum Sex {
    MALE(1,"男"),FEMALE(2,"女");
    private int id;
    private String name;
    private Sex(int id, String name) {
        this.id = id;
        this.name = name;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

# 3.1 EnumOrdinalTypeHandler

创建一张表来演示EnumOrdinalTypeHandler： 

```sql
CREATE TABLE `t_student` (
    `id` int(20) NOT NULL AUTO_INCREMENT COMMENT '编号',
    `cnname` varchar(60) NOT NULL COMMENT '学生姓名',
    `sex` tinyint(4) NOT NULL COMMENT '性别',
    `selfcard_no` int(20) NOT NULL COMMENT '学生证号',
    `note` varchar(1024) DEFAULT NULL COMMENT '备注',
    PRIMARY KEY (`id`)
);
```

这里sex字段类型为tinyint类型。 

Student实体类 :

```java
public class Student {
    private Long id;
    private String cnname;
    private Sex sex;
    private Integer selfcardNo;
    private String note;
......
}
```

定义一个interface： 

```java
public interface StudentMapper {

    List<Student> getAllStudent();

    int createStudent(Student stu);
}
```

对应的映射文件StudentMapper.xml： 

```xml
<!--和接口路径和名称保持一致，MyBatis会自动帮我们找到这个 Mapper-->
<mapper namespace="com.chp.mybatis.mapper.StudentMapper">
    <resultMap id="studentList" type="student">
        <id column="id" property="id"></id>
        <result column="cnname" property="cnname"></result>
        <result column="sex" property="sex" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler" />
        <result column="selfcard_no" property="selfcardNo"></result>
        <result column="note" property="note"></result>
    </resultMap>
    
    <select id="getAllStudent" resultMap="studentList" >
        <![CDATA[
            select * from t_student
        ]]>
    </select>

    <insert id="createStudent" parameterType="student">
        <![CDATA[
            insert into t_student(cnname,sex,selfcard_no,note)
            values (#{cnname},#{sex,typeHandler=org.apache.ibatis.type.EnumOrdinalTypeHandler},#{selfcardNo},#{note})
        ]]>
    </insert>
</mapper>
```

测试createStudent方法： 

```java
...
 StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
 Student stu = new Student();
 stu.setCnname("同学A");
 stu.setSelfcardNo(0001);
 stu.setSex(Sex.MALE);
 studentMapper.createStudent(stu);
...
```

查询数据库： 

```sql
mysql> select * from t_student;
+----+--------+-----+-------------+------+
| id | cnname | sex | selfcard_no | note |
+----+--------+-----+-------------+------+
|  1 | 同学A  |   0 |           1 | NULL |
+----+--------+-----+-------------+------+
```

可见EnumOrdinalTypeHandler已经将MALE转换为了MALE的下标了。 

测试getAllStudentTest方法：

```java
StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
List<Student> students = studentMapper.getAllStudent();
System.out.println(students);
```

测试结果：

```
[Student{id=1, cnname='同学A', sex=MALE, selfcardNo=1, note='null'}]
```

# 3.2 EnumTypeHandler

为了演示EnumTypeHandler ，借用note字段当做sex使用：

```java
private Sex note;
```

在映射文件添加typeHandler：

```xml
<!--和接口路径和名称保持一致，MyBatis会自动帮我们找到这个 Mapper-->
<mapper namespace="com.chp.mybatis.mapper.StudentMapper">
    <resultMap id="studentList" type="student">
        <id column="id" property="id"></id>
        <result column="cnname" property="cnname"></result>
        <result column="sex" property="sex" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler" />
        <result column="selfcard_no" property="selfcardNo"></result>
        <result column="note" property="note" typeHandler="org.apache.ibatis.type.EnumTypeHandler"></result>
    </resultMap>
    
    <select id="getAllStudent" resultMap="studentList" >
        <![CDATA[
            select * from t_student
        ]]>
    </select>

    <insert id="createStudent" parameterType="student">
        <![CDATA[    
            insert into t_student(cnname,sex,selfcard_no,note)
            values (#{cnname},#{sex,typeHandler=org.apache.ibatis.type.EnumOrdinalTypeHandler},#{selfcardNo},
            #{note,typeHandler=org.apache.ibatis.type.EnumTypeHandler})
        ]]>
    </insert>
</mapper>
```

测试createStudent方法： 

```java
StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
Student stu = new Student();
stu.setCnname("同学A");
stu.setSelfcardNo(0001);
stu.setSex(Sex.MALE);
stu.setNote(Sex.FEMALE);
studentMapper.createStudent(stu);
```

查询数据库： 

```sql
mysql> select * from t_student;
+----+--------+-----+-------------+--------+
| id | cnname | sex | selfcard_no | note   |
+----+--------+-----+-------------+--------+
|  1 | 同学A  |   0 |           1 | NULL   |
|  2 | 同学A  |   0 |           1 | FEMALE |
+----+--------+-----+-------------+--------+
```

可见EnumTypeHandler保存的是枚举字符串。 



[源码地址](https://github.com/ChaoHappy/MybatisAll/tree/master/03.Mybatis-TypeHandler)

