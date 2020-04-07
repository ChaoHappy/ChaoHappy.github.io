---
layout:     post
title:      SpringBoot2.0教程——基础配置
subtitle:   
date:       2020-04-07
author:     ChaoHappy
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - SpringBoot
    - SpringBoot2.0教程
---

# 1. 定制Banner

Spring Boot项目在启动的时候会有一个默认的启动图案：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.4.RELEASE)
```

我们可以把这个图案修改为自己想要的。在src/main/resources目录下新建banner.txt文件，然后将自己的图案黏贴进去即可。ASCII图案可通过网站http://www.network-science.de/ascii/一键生成，比如输入ChaoHappy生成图案后复制到banner.txt，启动项目，eclipse控制台输出如下：

> 我本地eclipse的banner.txt文件报错"expecting a 'map' but found a 'scalar' eclipse"，选择文件并执行Window->Show View->Problems ，右键单击“ Problems视图中的错误并将其删除即可，目前没有发现问题。

```
  _____ _                 _    _                         
 / ____| |               | |  | |                        
| |    | |__   __ _  ___ | |__| | __ _ _ __  _ __  _   _ 
| |    | '_ \ / _` |/ _ \|  __  |/ _` | '_ \| '_ \| | | |
| |____| | | | (_| | (_) | |  | | (_| | |_) | |_) | |_| |
 \_____|_| |_|\__,_|\___/|_|  |_|\__,_| .__/| .__/ \__, |
                                      | |   | |     __/ |
                                      |_|   |_|    |___/ 
```

banner也可以关闭，在main方法中：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(Application.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

# 2. 全局配置文件

