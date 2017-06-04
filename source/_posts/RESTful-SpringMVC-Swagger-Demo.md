---
title: RESTful SpringMVC Swagger Demo
category: Technology
toc: true
date: 2016-09-18 21:38:37
tags: [RESTful]
---
SpringMVC框架搭建 RESTful 风格后端服务器示例
源码：[RESTful-SpringMVC-Swagger-Demo](https://github.com/ZongWenlong/RESTful-SpringMVC-Swagger-Demo)
# 简介：
## 涉及框架或数据库
- SpringMVC 框架
- Spring 框架
- MyBatis 框架
- Log4j 日志框架
- MySQL 数据库
- Redis 数据库
- Swagger-UI API接口管理工具

## Try it
1. 导入项目到`IDEA/Eclipse`中
2. 初始化数据库，导入`tools/mysql-init.sql`到MySQL数据库（本示例仅包含一张表，一条记录）
3. 配置MySQL和Redis，修改`src//main/resource/database.properties`
    - mysql.url
    - mysql.username
    - mysql.password
    - redis.host
4. 修改`src/main/webapp/WEB-INF/swagger-ui/index.html`
    ```
    url = "http://localhost:8888/api-docs";
    修改为：
    url = "http://ip:port/{项目名}/api-docs
    ```

5. 启动
6. 浏览器中查看`http://ip:port/{项目名}/swagger/index.html`页面显示如下： ![image](https://github.com/ZongWenlong/Demos/blob/master/images/swagger/api-examples.PNG)

# 具体说明：

## 请求响应过程
- 登录：请求中携带用户名密码，获取Token, 为简单起见本示例只要用户名和密码相同则验证通过
    - Token
        - 凭证信息，用户在之后的请求中用Token中的字符串代替密码供鉴权使用，并有一定的使用期限
        - 为加速查询，Token字符串存储在Redis中，存储格式为Key-Value（name-tokenStr）
- 鉴权：除登录外的其他操作原则上均需要经过鉴权才能使用系统的API
    - 本示例中利用拦截器`src/java/pers/well/interceptor/AuthorizationInterceptor`拦截请求，进行Token正确性验证
    - `AuthorizationInterceptor` 在`src/main/resource/spring-mvc.xml`中配置

## 接口说明
- 示例中有三个接口：/login, /student, token/student
- /login
    - 请求中携带用户名密码，获取Token
- /student
    - 无token示例，提供简单的id参数即可进行查询
- /token/student
    - 需要鉴权的接口示例，必须携带正确的token和username，否则鉴权失败

## 部分配置项说明
- `src/main/resource/`目录：

    - database.properties: 数据库相关配置
    - log4j.properties: log4j日志配置
    - mybatis-config.xml: mybatis分页插件等相关配置
    - spring.xml: redis等配置
    - spring-mvc.xml： springmvc配置
    - spring-mybatis.xml: spring,mybatis整合配置

## Swagger-UI说明
- SpringMVC项目中引入Swagger的主要步骤：
    - 添加个性化配置：`src/java/pers/well/config/SwaggerConfig.java`
    - Copy Swagger-UI界面部分[SWagger-UI](https://github.com/swagger-api/swagger-ui/tree/master/dist) 到`webapp/WEB-INF/swagger-ui`
    - Spring配置
    ```
    <bean class="pers.well.config.SwaggerConfig" />
    <mvc:resources mapping="/swagger/**" location="/WEB-INF/swagger-ui/"/>
    ```
    - Controller中可以自主添加配置，更好的显示API：
        - @API
        - @ApiOperation
        - @ApiModel
        - @ApiModelProperty

主要参考：
> 1. [HelloWorld-MVC-Swagger](https://github.com/albertchendao/demos/tree/master/java/spring/HelloWorld-MVC-Swagger)
> 2. [Swagger: make developers love working with your REST API](https://www.javacodegeeks.com/2013/10/swagger-make-developers-love-working-with-your-rest-api.html)
            

## 其他说明：
- MyBatisGeneratorConfig.xml MyBatis代码自动生成器配置，可以根据数据库表信息自动反向生成MyBatis Mapper等文件，详细说明见 [MyBatis Generator](http://generator.sturgeon.mopaas.com/index.html)
    
   
