---
layout:     post
title:      Spring整合ActiveMQ
subtitle:   
date:       2020-02-14
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - ActiveMQ
    - Spring
---

......

[代码地址](https://github.com/ChaoHappy/spring-learn/tree/master/spring-activemq )

# 1 Maven 配置

```java
    <!-- activemq对JMS的支持，整合Spring和Activemq -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jms</artifactId>
      <version>4.3.23.RELEASE</version>
    </dependency>
    <!-- activemq所需要的pool包配置 -->
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-pool</artifactId>
      <version>5.15.9</version>
    </dependency>
```

# 2 Spring 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd">

    <context:component-scan base-package="com.chaohappy"/>

    <!-- 配置生产者 -->
    <bean id="jmsFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <!-- 真正可以生产Connection的ConnectionFactory，由对应的JMS服务厂商提供 -->
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="tcp://localhost:61616" />
            </bean>
        </property>
        <property name="maxConnections" value="100"></property>
    </bean>

    <!-- 这是队列目的地，点对点的 -->
    <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="spring-active-queue" />
    </bean>

    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="jmsFactory" />
        <property name="defaultDestination" ref="destinationQueue"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>

</beans>
```

# 3 代码示例

## 3.1 队列

### 3.1.1 生产者

```java
@Service
public class SpringMQ_Produce {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext tx = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Produce produce = (SpringMQ_Produce)tx.getBean("springMQ_Produce");
        produce.jmsTemplate.send(new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {

                TextMessage textMessage = session.createTextMessage("***spring和ActiveMQ的整合case111***");
                return textMessage;
            }
        });
        System.out.println("******send task over");
    }
}
```

### 3.1.2 消费者

```java
@Service
public class SpringMQ_Consumer {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext tx = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Consumer consumer = (SpringMQ_Consumer)tx.getBean("springMQ_Consumer");
        String retValue = (String)consumer.jmsTemplate.receiveAndConvert();
        System.out.println("******消费者收到的消息："+retValue);
    }
}
```

## 3.2 主题

只需将Spring配置文件按照如下调整（队列改成主题），其他代码都无需调整

```xml
<!-- 这是主题 -->
<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
    <constructor-arg index="0" value="spring-active-topic" />
</bean>

<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
    <property name="connectionFactory" ref="jmsFactory" />
    <property name="defaultDestination" ref="destinationTopic"/>
    <property name="messageConverter">
        <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
    </property>
</bean>
```

## 3.3 监听器

只需要启动生产者，不需要启动消费者，自动会监听记录。

Spring配置文件

```xml
<!-- 配置监听器 -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
<property name="connectionFactory" ref="jmsFactory" />
<property name="destination" ref="destinationQueue"/>
<!-- 注入实现MessageListener 接口实现类 -->
<property name="messageListener" ref="myMessageListener" />
</bean>
```

生产者

```java
@Service
public class SpringMQ_Produce {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext tx = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Produce produce = (SpringMQ_Produce)tx.getBean("springMQ_Produce");
        produce.jmsTemplate.send(new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("***spring和ActiveMQ的整合case111***");
                return textMessage;
            }
        });
        System.out.println("******send task over");
    }
}
```

监听实现类

```java
@Component
public class MyMessageListener implements MessageListener {
    @Override
    public void onMessage(Message message) {
        if(message!=null && message instanceof TextMessage){
            TextMessage textMessage =(TextMessage)message;
            try {
                String text = textMessage.getText();
                System.out.println("****监听器收到消息："+text);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

