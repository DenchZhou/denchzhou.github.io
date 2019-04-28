---
layout: post
title: ssm项目配置xml使用spring session进行session共享
categories: Spring
description: 使用POI解析Excel表格的demo
keywords: SpringSession Spring ssm session
---

### 概述
session的基础知识就不再多说。
通常，我们会把一个项目部署到多个tomcat上，通过nginx进行负载均衡，提高系统的并发性。此时，就会存在一个问题。假如用户第一次访问tomcat1，并登陆保存了用户信息，但是下一次访问的时候，nginx让用户访问tomcat2，此时tomcat2中并没有用户的session信息，用户必须重新进行登录操作。这样会极大的破坏用户的体验。
对此，我们有两大类解决方案。一个是将nginx的负载均衡机制设为根据iphash，也就是用户每次保证能访问同一台tomcat，但是该方法也存在着弊端，当tomcat挂掉以后，用户必须重新登陆。第二种方法较为常用，就是通过某些方法进行session共享。常用的方法有使用spring session，如果项目中有用到shiro，可以通过重写AbstractSessionDAO，即重写shiro的session存储机制完成。
本文将先从spring session入手，完成session共享。由于最近接触的是一个老项目，使用ssm框架，本文基于xml的方式进行配置。熟悉spring boot的朋友可以直接到spring boot官网进行查看，配置较为简单。由于老项目的并发量并不是很高，因此本文使用spirng-session-jdbc来进行session共享。大型项目可选择使用非关系型数据库Redis等

### 代码地址
https://github.com/DenchZhou/ssm/tree/master/spring-session-jdbc
如果有用的话不妨给个star，感恩！

### 配置
#### Maven依赖
加入maven依赖  以下使用的是2.0以上的版本 如果在没有网的环境下记得添加其他相关依赖如spring-jdbc、spring-session-core、spring-context
```
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-jdbc</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
```
#### 数据库表
我们需要添加两张数据库表，spring session通过将httpsession序列化保存至数据库，需要session的时候再从数据库中取出并反序列化。因此我们需要保证数据库中存在保存session的表格。
```
/* 2.x版本使用的数据库表*/
CREATE TABLE SPRING_SESSION (
	PRIMARY_ID CHAR(36) NOT NULL,
	SESSION_ID CHAR(36) NOT NULL,
	CREATION_TIME BIGINT NOT NULL,
	LAST_ACCESS_TIME BIGINT NOT NULL,
	MAX_INACTIVE_INTERVAL INT NOT NULL,
	EXPIRY_TIME BIGINT NOT NULL,
	PRINCIPAL_NAME VARCHAR(100),
	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
 
CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);
 
CREATE TABLE SPRING_SESSION_ATTRIBUTES (
	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
	ATTRIBUTE_BYTES BLOB NOT NULL,
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
上面的数据库表适用于2.0以上的版本，总共有七个字段。如果maven依赖1.x版本使用的数据库表不一样。
```
/* 1.x版本使用的数据库表*/
CREATE TABLE `SPRING_SESSION` (
  `SESSION_ID` char(36) NOT NULL DEFAULT '',
  `CREATION_TIME` bigint(20) NOT NULL,
  `LAST_ACCESS_TIME` bigint(20) NOT NULL,
  `MAX_INACTIVE_INTERVAL` int(11) NOT NULL,
  `PRINCIPAL_NAME` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`SESSION_ID`) USING BTREE,
  KEY `SPRING_SESSION_IX1` (`LAST_ACCESS_TIME`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `SPRING_SESSION_ATTRIBUTES` (
  `SESSION_ID` char(36) NOT NULL DEFAULT '',
  `ATTRIBUTE_NAME` varchar(100) NOT NULL DEFAULT '',
  `ATTRIBUTE_BYTES` blob,
  PRIMARY KEY (`SESSION_ID`,`ATTRIBUTE_NAME`),
  KEY `SPRING_SESSION_ATTRIBUTES_IX1` (`SESSION_ID`) USING BTREE,
  CONSTRAINT `SPRING_SESSION_ATTRIBUTES_ibfk_1` FOREIGN KEY (`SESSION_ID`) REFERENCES `SPRING_SESSION` (`SESSION_ID`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### spring xml相关配置
```
<context:annotation-config/>
    <bean class="org.springframework.session.jdbc.config.annotation.web.http.JdbcHttpSessionConfiguration">
        <property name="tableName" value="spring_session"/>
        <property name="maxInactiveIntervalInSeconds" value="1800"/>
    </bean>
<!-- 配置事务管理器 -->
    <bean id="transactionManager"
     class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
```
使用context:annotation-config和JdbcHttpSessionConfiguration是因为Spring Session没有提供XML命名空间的支持。我们可以在配置相关的属性注入，如数据库表名，session保持的时间等等。然后就是事务管理器的配置，数据源的配置不再详细说明了。可以看最上面的项目源码。
#### web.xml配置过滤器
spring session通过自定义一个filter，通过filter职责链将用自己定义的request替换httpservletrequest，从而使用自己httpsession。因此我们必须配置一下Filter，并且最好把他放在最前面，使其优先执行。
```
  <filter>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

```
spring 配置读取我们的spring配置文件。
```
<servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-*.xml</param-value>
        </init-param>

    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!-- 默认匹配所有的请求 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
### 测试
我的项目中只写了一个简单的查找user并放入session中
```
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/findUser")
    @ResponseBody
    public String findUserById(HttpServletRequest request){
        UserEntity user = userService.findUserById(1);
        HttpSession session = request.getSession();
        session.setAttribute("user", user);
        return user.toString();
    }
}
```
可以通过数据库管理工具查看
![image.png](https://upload-images.jianshu.io/upload_images/14607771-261b47675b8a558e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14607771-e4f6ee478161a814.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
