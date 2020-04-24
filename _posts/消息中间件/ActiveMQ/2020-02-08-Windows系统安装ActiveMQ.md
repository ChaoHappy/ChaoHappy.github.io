---
layout:     post
title:      Windows系统安装ActiveMQ
subtitle:   
date:       2020-02-08
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - ActiveMQ
---

## 下载

<http://activemq.apache.org/components/classic/download/> 

## 安装

1. 将下载的压缩包解压到本地磁盘
2. 进入安装目录的\apache-activemq-5.15.11\bin\win64 文件夹下
3. 注册为windows服务
   1. 可以使用activemq.bat启动服务。但是关闭该窗口，服务停止。故需要将activemq注册为windows服务。 
   2. 使用 InstallService.bat注册服务，双击此文件即可在系统服务面板中找到服务名称为“ActiveMQ”的服务。

## 启动服务

1.  启动“ActiveMQ”服务
2.  输入地址<http://localhost:8161/>  
3.  如果出现ActiveMQ相关的页面则安装、启动成功。

说明：

​	进入控制台端口默认为8161，61616为默认对外服务端口。

​	当端口号冲突时，可以修改这两个端口号。进入conf目录下修改activemq.xml-修改里面的61616端口。修改	jetty.xml-修改里面的8161端口。

 

## 查看密码

ActiveMQ默认用户名密码为admin=admin,可以在conf/user.properties中找到。如下所示：

```properties
## ---------------------------------------------------------------------------
## Licensed to the Apache Software Foundation (ASF) under one or more
## contributor license agreements.  See the NOTICE file distributed with
## this work for additional information regarding copyright ownership.
## The ASF licenses this file to You under the Apache License, Version 2.0
## (the "License"); you may not use this file except in compliance with
## the License.  You may obtain a copy of the License at
##
## http://www.apache.org/licenses/LICENSE-2.0
##
## Unless required by applicable law or agreed to in writing, software
## distributed under the License is distributed on an "AS IS" BASIS,
## WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
## See the License for the specific language governing permissions and
## limitations under the License.
## ---------------------------------------------------------------------------

admin=admin
```

## 修改密码

修改activemq.xml，在broker标签内部添加配置如下： 

```
<plugins>
	<simpleAuthenticationPlugin>
		<users>
			<authenticationUser username="adm" password="adm" groups="users,admins"/>
		</users>
	</simpleAuthenticationPlugin>
</plugins>
```



