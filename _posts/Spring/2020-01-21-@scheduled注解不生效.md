---
layout:	post
title:	Scheduled注解不生效
subtitle:   
date:	2020-01-21
author:	BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Spring
    - scheduled
---

......

写了个定时任务，执行测试用例加载容器，等待定时任务触发，但是@scheduled注解不生效。

代码如下：

```java
@Component
public class ReconRetryTaskJobService {

    @Scheduled(fixedDelay = 500L)
    public void takeRetrySchedule(){
        AbstractRetryTask task = ReconciliationServiceImpl.RETRY_TASK_QUEUE.poll();
        Thread thread = new Thread(task);
        thread.start();
    }

}
```

以前一直都是这么写的，为什么不生效了呢？

经过网上查资料发现，原来是Spring版本差异造成的，以前我用的是Spring4.2.8，现在用的是4.3.25版本。

4.3.25版本的定时任务类需要增加@EnableScheduling注解，代码如下：

```java
@EnableScheduling
@Component
public class ReconRetryTaskJobService {

    @Scheduled(fixedDelay = 500L)
    public void takeRetrySchedule(){
        AbstractRetryTask task = ReconciliationServiceImpl.RETRY_TASK_QUEUE.poll();
        Thread thread = new Thread(task);
        thread.start();
    }

}
```

测试可以正常触发定时任务。