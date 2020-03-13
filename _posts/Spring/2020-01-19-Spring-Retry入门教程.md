---
layout:	post
title:	Spring-Retry 入门教程
subtitle:   
date:	2020-01-19
author:	BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Spring
    - Spring Batch
---

[代码地址](https://github.com/ChaoHappy/spring-learn/tree/master/spring-retry)

# spring-retry
# 1. 概述
Spring Retry提供了自动重新调用失败操作的能力。这对于错误可能是暂时性的（如暂时性的网络故障）很有帮助。Spring Retry提供了对流程和基于策略的行为的声明性控制，易于扩展和定制。

# 2. Maven依赖
首先，将依赖项添加到pom.xml中：
 ```xml
<properties>
        <spring.version>4.3.25.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>

        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
 ```

# 3. 启动Spring Retry （注解方式）
要在应用程序中启用Spring Retry，我们需要将@enablerery注释添加到实现类上：
```java
@EnableRetry
@Service
public class MyServiceImpl implements MyService{

    public void retryService(String sql) throws SQLException {
        System.out.println("retryService");
        throw new SQLException();
    }

    public void recover(SQLException e, String sql) {
        System.out.println("recover");
    }
}
```
# 4. 使用Retry注解

 我们可以使用注释进行方法调用，以便在失败时重试。 

## 4.1 @Retryable

 要向方法添加重试功能，可以使用@Retryable： 

 ```java
 @Service
 public interface MyService {
     @Retryable(
       value = { SQLException.class }, 
       maxAttempts = 2,
       backoff = @Backoff(delay = 5000))
     void retryService(String sql) throws SQLException;
     ...
 }
 ```
这里，重试行为是使用@Retryable的属性定制的。在本例中，仅当方法引发SQLException时，才会尝试重试。最多可重试2次，延迟5000毫秒。 

如果使用@Retryable时没有任何属性，如果方法因异常而失败，则将尝试最多三次重试，延迟一秒。 

Retryable注解参数：

- value：指定发生的异常进行重试
- include：和value一样，默认空，当exclude也为空时，所有异常都重试
- exclude：指定异常不重试，默认空，当include也为空时，所有异常都重试
- maxAttemps：重试次数，默认3
- backoff：重试补偿机制，默认没有 

@Backoff 注解 重试补偿策略：

- 不设置参数时，默认使用FixedBackOffPolicy（指定等待时间），重试等待1000ms
- 设置delay,使用FixedBackOffPolicy（指定等待- - 设置delay和maxDealy时，重试等待在这两个值之间均态分布
- 设置delay、maxDealy、multiplier，使用 ExponentialBackOffPolicy（指数级重试间隔的实现 ），multiplier即指定延迟倍数，比如delay=5000l,multiplier=2,则第一次重试为5秒，第二次为10秒，第三次为20秒

## 4.2 @Recover

@Recover注释用于在@Retryable方法因指定的异常而失败时定义单独的恢复方法： 

```java
@Service
public interface MyService {
    ...
    @Recover
    void recover(SQLException e, String sql);
}
```

因此，如果retryService（）方法抛出一个SQLException，将调用recover（）方法。合适的恢复处理程序的第一个参数类型为Throwable（可选）。从失败方法的参数列表中按与失败方法相同的顺序填充后续参数，并使用相同的返回类型。 

- 当重试次数耗尽依然出现异常时，执行此异常对应的@Recover方法。
- 异常类型需要与Recover方法参数类型保持一致，
- recover方法返回值需要与重试方法返回值保证一致

## 4.3 总结
虽然注解使用简便，但是可配置的东西太少，如果想使用spring-retry强大的策略机制，并需定制化RetryTemplate。
RetryTemplate 对象可以设置重试策略、补偿策略、重试监听等属性。

# 5. RetryTemplate

## 5.1 RetryOperations
Spring Retry提供RetryOperations接口，该接口提供一组execute（）方法：

 ```java
public interface RetryOperations {
   <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback) throws E;
   
   <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback)
           throws E;
   
   <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RetryState retryState)
           throws E, ExhaustedRetryException;
   
   <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback,
           RetryState retryState) throws E;
}
 ```
 RetryCallback是execute（）的一个参数，它是一个接口，允许插入失败时需要重试的业务逻辑：
 ````java
public interface RetryCallback<T> {
    T doWithRetry(RetryContext context) throws Throwable;
}

 ````
RecoveryCallback : 当结束时会调用recover（）方法，RetryOperations可以将控制权传递给另一个回调，称为RecoveryCallback。

```java
@Override
public Object recover(RetryContext context) throws Exception {
    System.out.println("testRetryTemplate=========：recover");
    return null;
}
```



## 5.2 RetryTemplate使用

 RetryTemplate是RetryOperations的实现。让我们简单使用下吧：
 ```java
 @Test
     public void testRetryTemplate() throws Throwable {
         RetryTemplate retryTemplate = new RetryTemplate();
         //设置重试策略 最大重试次数2次（什么时候重试）,默认遇到Exception异常时重试。
         SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
         retryPolicy.setMaxAttempts(2);
         retryTemplate.setRetryPolicy(retryPolicy);
 
         // 设置重试间隔时间 3S
         FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
         fixedBackOffPolicy.setBackOffPeriod(3000l);
         retryTemplate.setBackOffPolicy(fixedBackOffPolicy);
 
         //使用重试
         retryTemplate.execute(new RetryCallback<Object, Throwable>() {
 
             @Override
             public Object doWithRetry(RetryContext context) throws Throwable {
                 myService.retryService("testRetryTemplate");
                 return null;
             }
         }, new RecoveryCallback<Object>() {
 
             @Override
             public Object recover(RetryContext context) throws Exception {
                 System.out.println("testRetryTemplate=========：recover");
                 return null;
             }
         });
     }
 ```



# 6. XML配置

个人认为 xml配置以后基本会被Spring Boot自动配置所取代，所以这块内容不进行深入学习，后续有需要会再更新。

 

 

 

