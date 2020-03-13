---
layout:	post
title:	《Spring实战3》第九章 保护Spring应用
subtitle:   
date:	2019-12-02
author:	BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Spring
    - 读书笔记
    - 《Spring实战3》
    - Spring
---

.....



Spring Security是一种基于SpringAOP和Servlet过滤器实现的安全框架。

# Spring Security介绍
Spring Security是为基于Spring的应用程序提供声明式安全保护的安全框架。Spring Security提供了完整的安全性解决方案，它能够在Web请求级别和方法调用级别处理身份验证和授权。因为基于Spring框架，所以Spring Security充分利用了依赖注入和面向切面的技术。
Spring Security从两个角度来解决安全性问题。它使用Servlet过滤器保护Web请求并限制URL级别的访问，也可以使用SpringAOP保护方法调用——借助与对象代理和使用通知，能够确保只有具备适当权限的用户才能访问安全保护的方法。
## Spring Security 起步
Spring Security3.0分为了8个模块。应用程序类路径下至少要包含核心和配置两个模块。Spring Securiy用于保护Web需要添加Web模块。Spring Security的JSP标签库需要添加Tag Library模块。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202210727735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoYW9IYXBweQ==,size_16,color_FFFFFF,t_70)
## 使用Spring Security配置命名空间

```
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
	xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    					http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                        http://www.springframework.org/schema/security 
                        http://www.springframework.org/schema/security/spring-security.xsd">
</beans:beans>
```
# 搭建简单登录&注销demo
## 配置maven
```
<dependency>
	      <groupId>org.springframework.security</groupId>
	      <artifactId>spring-security-web</artifactId>
	      <version>4.1.0.RELEASE</version>
	    </dependency>
	    <dependency>
	      <groupId>org.springframework.security</groupId>
	      <artifactId>spring-security-config</artifactId>
	      <version>4.1.0.RELEASE</version>
</dependency>
```

## 代理Servlet 过滤器
Spring Security借助一系列Servlet过滤器来提供各种安全性功能。我们只需在应用的web.xml中配置一个过滤器，如下所示：

```
<!-- Spring Security -->
  <filter>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
 
  <filter-mapping>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
```

## 配置Spring Security 
spring-security.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
	xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    					http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                        http://www.springframework.org/schema/security 
                        http://www.springframework.org/schema/security/spring-security.xsd">
	
	<http auto-config="true" use-expressions="false">
		<intercept-url pattern="/admin**" access="ROLE_ADMIN"/>
		<form-login 
		    login-page="/login"
		    login-processing-url="/j_spring_security_check"
		    default-target-url="/welcome" 
			authentication-failure-url="/login?error" 
			username-parameter="username"
			password-parameter="password" />
		<logout logout-url="/j_spring_security_logout" logout-success-url="/login?logout" />
		<!-- enable csrf protection -->
		<csrf/>
	</http>

	<authentication-manager>
		<authentication-provider>
		  <user-service>
			<user name="user1" password="123456" authorities="ROLE_USER" />
			<user name="admin" password="123456" authorities="ROLE_ADMIN" />
		  </user-service>
		</authentication-provider>
	</authentication-manager>
</beans:beans>
```
**form-login属性详解**
form-login是spring security命名空间配置登录相关信息的标签,它包含如下属性：

>  - login-page 自定义登录页url,默认为/login
>  - login-processing-url 登录请求拦截的url,也就是form表单提交时指定的action
>  - default-target-url 默认登录成功后跳转的url
>  - always-use-default-target 是否总是使用默认的登录成功后跳转url
>  - authentication-failure-url 登录失败后跳转的url
>  - username-parameter 用户名的请求字段 默认为userName
>  - password-parameter 密码的请求字段 默认为password
>  - authentication-success-handler-ref 指向一个AuthenticationSuccessHandler用于处理认证成功的请求,不能和default-target-url还有always-use-default-target同时使用
>  - authentication-success-forward-url 用于authentication-failure-handler-ref
>  - authentication-failure-handler-ref 指向一个AuthenticationFailureHandler用于处理失败的认证请求
>  - authentication-failure-forward-url 用于authentication-failure-handler-ref
>  - authentication-details-source-ref 指向一个AuthenticationDetailsSource,在认证过滤器中使用

**logout 属性详解**
当我们指定了http元素的auto-config属性为true时logout定义是会自动配置的，此时我们默认退出登录的URL为“/j_spring_security_logout”，可以通过logout元素的logout-url属性来改变退出登录的默认地址。

> - invalidate-session 表示是否要在退出登录后让当前session失效，默认为true。
> - delete-cookies 指定退出登录后需要删除的cookie名称，多个cookie之间以逗号分隔。
> - logout-success-url 指定成功退出登录后要重定向的URL。需要注意的是对应的URL应当是不需要登录就可以访问的。
> - success-handler-ref 指定用来处理成功退出登录的LogoutSuccessHandler的引用。

**intercept-url 属性详解**
**指定拦截的url**：通过pattern指定当前intercept-url定义应当作用于哪些url。
**指定访问权限**： 可以通过access属性来指定intercept-url对应URL访问所应当具有的权限。access的值是一个字符串，其可以直接是一个权限的定义，也可以是一个表达式。常用的类型有简单的角色名称定义，多个名称之间用逗号分隔。其中必须要以ROLE_为前缀。
```
<intercept-url pattern="/admin**" access="ROLE_ADMIN,ROLE_USER"/>
```
还支持Spring表达式，必须将< http > 的use-expressions设置为true
```
<intercept-url pattern="/admin**" access="hasRole(ROLE_ADMIN)"/>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/201912142116182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoYW9IYXBweQ==,size_16,color_FFFFFF,t_70)
**指定访问协议**: 如果你的应用同时支持Http和Https访问，且要求某些URL只能通过Https访问，这个需求可以通过指定intercept-url的requires-channel属性来指定。requires-channel支持三个值：http、https和any。any表示http和https都可以访问。


## 配置控制器
LoginController.java

```
@Controller
public class LoginController {
	@RequestMapping(value="/login")
	public ModelAndView showHomePage(@RequestParam(value = "error", required = false) String error,
			@RequestParam(value = "logout", required = false) String logout) {
		ModelAndView model = new ModelAndView();
		if (error != null) {
			model.addObject("error", "无效的用户名和密码！");
		}
		if (logout != null) {
			model.addObject("msg", "您已成功注销。");
		}
		model.setViewName("login");
		return model;
	}
	
	@RequestMapping(value="/success")
	public String showSuccessPage() {
		return "success";
	}
	
	@RequestMapping(value="/welcome")
	public String showWelcomePage() {
		return "hello";
	}
	
	@RequestMapping(value="/admin")
	public String showAdminPage() {
		return "admin";
	}
}
```
## 前台页面代码
login.jsp

```
<body>
<div id="login-box">
	<h3>Spring Security 欢迎进入</h3>	
	<c:if test="${not empty error }">
		<div class="error">${error}</div>
	</c:if>
	<c:if test="${not empty msg }">
		<div class="msg">${msg}</div>
	</c:if>
	<form action='<c:url value='/j_spring_security_check' />' method="post">
		<table>
			<tr>
				<td>用户名：</td>
				<td>
					<input type="text" name="username"/>
				</td>
			</tr>
			<tr>
				<td>密码：</td>
				<td>
					<input type="password" name="password"/>
				</td>
			</tr>
			<tr>
				<td colspan="2" align="center">
					<input type="submit" value="登录" />
				</td>
			</tr>
		</table>
			<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" />
	</form>
</div>
</body>
```
hello.jsp

```
<body>
	<h1>欢迎进入 Spring Security</h1>
	<c:url value="/j_spring_security_logout" var="logoutUrl" />
	<form action="${logoutUrl}" method="post" id="logoutForm">
		<input type="hidden" name="${_csrf.parameterName}"
			value="${_csrf.token}" />
	</form>
	<a href="javascript:formSubmit()">注销登录</a>
	<a href="<c:url value="/admin" />">后台管理</a>
</body>
```
admin.jsp

```
<body>
	<h1>欢迎进入：后台管理页面</h1>
</body>
```
# 保护视图级别的元素
为了支持视图级别的安全性，Spring Security提供了一个JSP标签库。这个标签库很小且只包含3个标签，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191214214806625.png)
为了使用JSP标签库，需要在对应JSP中声明如下：

```
<%@taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```
此外还需要加入以下依赖

```
<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-taglibs</artifactId>
			<version>4.1.0.RELEASE</version>
			<type>jar</type>
			<scope>compile</scope>
</dependency>
```
## 访问认证信息的细节
显示认证用户的用户名
```
<h1>你好，<security:authentication property="principal.username"/></h1>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191215223808946.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoYW9IYXBweQ==,size_16,color_FFFFFF,t_70)
## 根据权限渲染
< security:authorize>JSP标签能够根据用户授予的权限有条件地渲染页面的部分内容。

```
<security:authorize access="hasRole('ROLE_ADMIN')"><a>admin 用户才可以看到</a></security:authorize>

<security:authorize url="/admin**"><a>admin 用户才可以看到</a></security:authorize>
```
注意：在上下文中需要加载如下bean

```
<beans:bean id="webexpressionHandler" class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
	</beans:bean>
```

# 认证用户

 - 内存（基于Spring配置）用户存储库
 - 基于JDBC的用户存储库
 - 基于LDAP的用户存储库
 - OpenID分散式的用户身份识别系统
 - 中心认证服务（CAS）
 - X.509证书
 - 基于JAAS的提供者
## 配置内存用户存储库
在Spring配置中声明用户的详细信息，代码如下：

```
<user-service>
			<user name="user1" password="123456" authorities="ROLE_USER" />
			<user name="admin" password="123456" authorities="ROLE_ADMIN" />
</user-service>
```
用户信息声明在< user-service>之中，每个能登录应用程序的用户只有一个< user>元素，属性name和password分别指定了登录名和密码，authorities属性用于设置逗号分隔权限列表——即允许用户做的事情。
用户服务准备好后，剩下的事情就是将其装配到Spring Security的认证管理器中：

```
<authentication-manager>
		<authentication-provider user-service-ref="userService" />
	</authentication-manager>
```

## 基于数据库进行认证
用关系型数据库中存储用户信息：

```
<!-- 基于数据库进行认证 -->
	<jdbc-user-service id="jdbcUserService" data-source-ref="dataSource"
		users-by-username-query="SELECT username,password,true FROM user WHERE username=?"
		authorities-by-username-query="SELECT username,authority FROM user WHERE username=?"/>
	
	<authentication-manager erase-credentials="false">
		<authentication-provider user-service-ref="jdbcUserService" />
	</authentication-manager>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216210455832.png)
## 基于LDAP的用户存储库
目前用不上这块后续学习更新。。。
## 启用remember-me功能

```
< http>
	...
	<remember-me key="spitter" token-validity-seconds="241920"/>
< /http >
```

# 保护方法调用
目前用不上这块后续学习更新。。。