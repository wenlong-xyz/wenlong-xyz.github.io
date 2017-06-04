---
title: Web开发中的一些基本概念问题
category: Technology
toc: true
date: 2016-12-18 20:37:51
tags:
---
&emsp;&emsp;个人以为，学习新知识分为三个阶段：初识，通过书籍、网络学习基础入门知识；实践，通过实际应用印证、加深对知识的理解；探究，研究背后原理机制，做到知其然也知其所以然。之前曾从事过一些Java Web开发相关的工作，但一直都没有静下来细细研究其背后机理，现在希望通过整理一些问题看看其背后的故事。

# 基本概念
## Servlet VS JSP
Servlet生命周期：
1. **实例化**
    * 请求到达容器时，容器查找该对象是否存在，不存在则创建；
    * 如果web.xml中有对Servlet配置load-on-startup，则容器在启动时会创建servlet
2. **初始化**
    * 为Servlet分配资源，调用init方法
3. **就绪/调用**
    * 调用servlet对象的service方法，然后依据请求方式调用doGet, doPost等方法（这些方法需要重载）
4. **销毁**
    * 容器依据自身算法，调用destroy()方法销毁servlet对象，释放资源   
    * 通常在关闭Web应用时销毁Servlet实例
init与destroy只会被调用一次，但service会调用多次。
Servlet与JSP的区别：
* JSP 编译之后变成Servlet。
* JSP面向页面显示(java in html)，servlet面向逻辑控制(html in java)
* JSP中共有内置对象（request,respose，session等），但必须通过HttpServletRequest，HttpServletResponse，HttpServlet等获得，Servlet中没有。

参考：[jsp、Servlet相关知识](http://www.cnblogs.com/0201zcr/p/4693365.html)

## 过滤器、拦截器与监听器
* 过滤器(Filter): 对一堆东西进行筛选过滤，过滤器关心的是待筛选的对象，Servlet中有规范规定，应用：编码过滤器
```xml
<!-- web.xml中配置 --> 
<filter>
    <filter-name>SpringEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>SpringEncodingFilter</filter-name>
    <url-pattern>/</url-pattern>
</filter-mapping>
```
* 拦截器(Interceptor): 对流程的干预，对一个操作增加前置任务或者后置任务，Structs2, SpringMVC中的组件，面向切面编程，关注的是执行流程。应用：日志记录、权限检查等
```xml
<!-- ApplicationContext.xml中配置 -->  
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/status/**" />
        <bean class="a.b.c.DevAppAuthorizationInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```
* 监听器(Listerner): 当一个事件发生时，希望获取时间发生的详细信息，并进行一些操作。应用：SpringMVC中，启动时，自动装配ApplicationContext
```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
参考：[过滤器、拦截器和监听器的区别](https://www.zhihu.com/question/35225845)

## Cookie/Session/ServletContext
&emsp;&emsp;Cookie与Session都是常用的Web会话跟踪技术，用来跟踪每个用户的整个会话。而ServletContext则是面向的是整个Web应用。
* **Cookie**：存储在客户端（浏览器）中，Key-Value结构，客户端的每次请求都会携带给服务端。常见应用：“记住密码”，免密码登录；用户习惯分析，广告推送等。一般要求数据量比较小，因为每次请求都会携带。
    - 可设置有效期
    - 需要浏览器支持（浏览器可以禁用）
    - 不可跨域名（二级域名之间通过设置Domain参数后可以使用）
* **Session**: 存储在服务端，每个客户端对应一个Session对象，可以通过请求中携带的Session ID表示来区分客户，每个Session也是Key-Value结构。对于SessionID可以通过Cookie保存，比如HttpServletRequest就可以为session自动创建sessionid保存在cookie中（key为JSESSIONID）。也可以通过URL重写等方式，将Sessionid放在URL中。主要应用，购物车；限制用户对某些特定界面的访问；移动端token等。
    - 可设置Session有效期
    - 不易太大，因为Session一般存储在服务器内存中，session太大容易造成服务器内存溢出。当然现在比较流行的是使用Redis等内存数据库来实现session的相关功能。
* **ServletContext**: 存储在服务器，一个WEb应用只会有一个ServletContext,代表当前Web应用，Key-Value结构，用来存储Web应用的公共信息。应用：网站在线用户统计，流量统计等

参考：
1. [Cookie/Session机制详解](http://blog.csdn.net/fangaoxin/article/details/6952954)
2. [讲透Session、Cookie和ServletContext](http://blog.csdn.net/happyaaaaaaaaaaa/article/details/51303918)

## Post请求中四种常见的数据提交方式
* **application/x-www-form-urlencoded**: 浏览器原生<form>表单，如果不设置enctype属性，提交数据被转化为key-value形式
```http
POST  HTTP/1.1
Host: www.expamle.com
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

key1=value1&key2=value2

```
* **multipart/form-data**: 使用表单上传文件时，必须让<form>表单的enctype等于multipart/form-data
```html
    <form action="save" method="post" enctype="multipart/form-data"></form>
```
    Body中各部分以boundary分割请求示例：
    ```http
    POST  HTTP/1.1
    Host: www.expamle.com
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
    Cache-Control: no-cache

    ------WebKitFormBoundary7MA4YWxkTrZu0gW
    Content-Disposition: form-data; name="key1"

    value1
    ------WebKitFormBoundary7MA4YWxkTrZu0gW
    Content-Disposition: form-data; name="key2"

    value2
    ------WebKitFormBoundary7MA4YWxkTrZu0gW
    Content-Disposition: form-data; name="fileUpload1"; filename="net.PNG"
    Content-Type: image/png

    PNG ... content of net.PNG ...
    ------WebKitFormBoundary7MA4YWxkTrZu0gW--
    ```
* **application/json**: 数据以json字符串的方式提交，可以提交复杂的结构化数据，特别适合RESTful的接口。
```http
POST  HTTP/1.1
Host: www.expamle.com
Content-Type: application/json
Cache-Control: no-cache

{"name":"testUser","age":"25"}
```
* **text/xml**: 数据以xml的格式提交，常见的应用由XML-RPC
```http
<?xml version="1.0"?>
<methodCall>
    <methodName>examples.getStateName</methodName>
    <params>
        <param>
            <value><i4>41</i4></value>
        </param>
    </params>
</methodCall>
```

Postman测试工具中对上述四种数据提交方式均支持：
* form-data: multipart/form-data
* x-www-form-urlencoded: multipart/form-data
* raw: 在header中添加相应的数据类型即可，application/json;text/xml

参考：
1. [四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)
2. [postman中 form-data、x-www-form-urlencoded、raw、binary的区别](http://blog.csdn.net/wangjun5159/article/details/47781443)

# 实现机制
## Web.xml的加载过程
当我们去启动一个WEB项目时，容器包括（JBoss、Tomcat等）首先会读取项目web.xml配置文件里的配置。配置的读取与解析过程如下：
1. 读取<contxt-param></contxt-param>和<listener></listener>节点
    * 容器创建一个servletContext, 整个web项目所有部分都会共享这个上下文
    * 容器以<context-param></context-param>的name作为键，value作为值，将其转化为键值对，存入ServletContext。
    * 容器创建<listener></listener>中的类实例，根据配置的class类路径<listener-class>来创建监听，在ContextLoaderListener监听中会有contextInitialized(ServletContextEvent args)初始化方法，启动Web应用时，系统调用Listener的该方法，载入IoC容器。ContextLoaderListener类继承了ContextLoader，在初始化Context的过程中，调用ContextLoader的initWebApplicationContext方法初始化WebApplicationContext。WebApplicationContext是一个接口，Spring默认的实现类为XmlWebApplicationContext，进行IoC容器的初始化。其中Bean的定义配置是<context-param>指定的
    
    ```java
    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }
    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {   
        // 部分代码   
        if(servletContext.getAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
                throw new IllegalStateException("...");
        }else{
            if(this.context == null) {
                this.context = this.createWebApplicationContext(servletContext);
            }
        }
        // 将 WebApplicationContext 存储在 ServletContext中
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
    }
    ```


2. 读取<filter></filter>节点，实例化过滤器
3. 加载并初始化Servlet实例，并为其初始化自己的IOC context(类型也是XmlWebApplicationContext)，用于持有相关的Bean，将WebApplicationContext设置为他的父容器。初始完自己的IoC Context后，会将其保存到ServleContext中。每个Servlet都有自己Bean的空间，并且共享着key为WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE的WebApplicationContext。初始化策略,包括控制器映射，视图解析等：
    ```java
    protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }
    ```

总体顺序：context-param >> listener >> fileter >> servlet
参考： 
1. [Spring IoC Context启动过程解析](http://tinylcy.me/2016/06/21/Spring-IoC-Context%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/)
2. [看看Spring的源码(一)——Bean加载过程](https://my.oschina.net/gongzili/blog/304101)
