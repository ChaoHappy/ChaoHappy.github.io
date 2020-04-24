---
layout:     post
title:      SpringBoot整合ActiveMQ
subtitle:   
date:       2020-02-15
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - ActiveMQ
    - SpringBoot
---

[代码地址](https://github.com/ChaoHappy/spring-boot-learn/tree/master/spring-boot-activemq  )

# 1 Maven配置

```xml
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

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.1.5.RELEASE</version>
        </dependency>
    </dependencies>
```

# 2 队列——代码示例

## 2.1 核心配置文件

application.yml

```yaml
server:
  port: 8080
spring:
  activemq:
    broker-url: tcp://localhost:61616 # 自己的MQ服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: false   # false = Queue   true = Topic

# 定义队列名称
mqqueue: boot-activemq-queue
```

## 2.2 配置Bean

```java
@Component
public class ConfigBean {

    @Value("${myqueue}")
    private String myqueue;

    @Bean
    public Queue queue(){
        return new ActiveMQQueue(myqueue);
    }
}
```

## 2.3 生产者代码

```java
@Component
public class Queue_Produce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Queue queue;

    public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(queue, UUID.randomUUID().toString().substring(0,8));
        System.out.println("*****");
    }
}
```

## 2.4 主程序 

```java
@SpringBootApplication
@EnableJms
public class SpringBootActivemqApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootActivemqApplication.class, args);
    }
}
```

## 2.5 测试

```java
@SpringBootTest(classes = SpringBootActivemqApplication.class)
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class ActiveMQTest {
    @Autowired
    private Queue_Produce queueProduce;

    @Test
    public void testSend() {
        queueProduce.produceMsg();
    }
}
```

## 2.6 扩展——定时推送消息

主程序增加注解@EnableScheduling

```
 @Scheduled(fixedDelay = 3000L)
 public void produceMsgScheduled(){
 produceMsg();
 }
```

## 2.7 消费者——监听器

```java
@Component
public class Queue_Consumer {

    @JmsListener(destination = "${myqueue}")
    public void receive(TextMessage textMessage) throws JMSException {
        System.out.println("*****消费者收到消息："+textMessage.getText());
    }
}
```

# 3 主题——代码示例

## 3.1 核心配置文件

application.yml

```yaml
server:
  port: 8080
spring:
  activemq:
    broker-url: tcp://localhost:61616 # 自己的MQ服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: true   # false = Queue   true = Topic

# 定义队列名称
mytopic: boot-activemq-topic
```

## 3.2 配置Bean

```java
@Component
public class ConfigBean {

    @Value("${mytopic}")
    private String mytopic;

    @Bean
    public Topic topic(){
        return new ActiveMQTopic(mytopic);
    }
}
```

## 3.3 生产者代码

```java
@Component
public class Topic_Produce {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    @Scheduled(fixedDelay = 3000L)
    public void produceTopic(){
        jmsMessagingTemplate.convertAndSend(topic,"************主题消息："+ UUID.randomUUID().toString().substring(0,8));
    }
}
```

## 3.4 消费者

```java
@Component
public class Topic_Consumer {

    @JmsListener(destination = "${mytopic}")
    public void receive(TextMessage textMessage) throws JMSException {
        System.out.println("消费者收到消息："+textMessage.getText());
    }
}
```

