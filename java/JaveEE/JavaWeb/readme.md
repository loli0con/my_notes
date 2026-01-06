# JavaWeb
JavaEE最核心的组件就是基于Servlet标准的Web服务器，开发者编写的应用程序是基于Servlet API并运行在Web服务器内部的：
```
┌─────────────┐
│┌───────────┐│
││ User App  ││
│├───────────┤│
││Servlet API││
│└───────────┘│
│ Web Server  │
├─────────────┤
│   JavaSE    │
└─────────────┘
```

## JavaWeb服务器
JavaWeb服务器：
* [Tomcat](tomcat.md)
  * [Tomcat目录结构](tomcat.md#Tomcat目录结构)
  * [终端日志中文乱码问题](tomcat.md#终端日志中文乱码问题)
  * [WEB项目的标准发布结构](tomcat.md#WEB项目的标准发布结构)
  * [WEB项目部署的方式](tomcat.md#WEB项目部署的方式)
    * [webapps真实目录发布](tomcat.md#webapps真实目录发布)
    * [虚拟目录发布](tomcat.md#虚拟目录发布)
    * [独立xml文件虚拟目录发布](tomcat.md#独立xml文件虚拟目录发布)

## JavaWeb三大组件
![tomcat+20210722094529](https://raw.githubusercontent.com/loli0con/picgo/master/images/tomcat%2B20210722094529.png%2B2021-07-22-09-45-31)

JavaWeb三大组件：
* [Servlet程序](servlet.md)
  * [Servlet的继承体系](servlet.md#Servlet的继承体系)
  * [Servlet的开发示例](servlet.md#Servlet的开发示例)
  * [Servlet的开发流程](servlet.md#Servlet的开发流程)
  * [Servlet的两种注册方式](servlet.md#Servlet的两种注册方式)
    * [通过xml注册Servlet](servlet.md#通过xml注册Servlet)
    * [通过注解注册Servlet](servlet.md#通过注解注册Servlet)
  * [Servlet的映射规则](servlet.md#Servlet的映射规则)
  * [Servlet的生命周期](servlet.md#Servlet的生命周期)
  * [Servlet的运行原理](servlet.md#Servlet的运行原理)
  * [ServletConfig](servlet.md#ServletConfig)
  * [HttpServletRequest](servlet.md#HttpServletRequest)
  * [HttpServletResponse](servlet.md#HttpServletResponse)
  * [转发和重定向](servlet.md#转发和重定向)
  * [三大作用域](servlet.md#三大作用域)
    * [通用API](servlet.md#通用API)
    * [ServletContext全局上下文域](servlet.md#ServletContext全局上下文域)
    * [Cookie技术](servlet.md#Cookie技术)
    * [Session会话域](servlet.md#Session会话域)
* [~~Jsp动态网页~~](jsp.md)
* [Filter过滤器](filter.md)
  * [Filter的执行流程](filter.md#Filter的执行流程)
  * [Filter的注册方式](filter.md#Filter的注册方式)
    * [通过注解注册Filter](filter.md#通过注解注册Filter)
    * [通过xml注册Filter](filter.md#通过xml注册Filter)
  * [Filter的映射规则](filter.md#Filter的映射规则)
  * [Filter的生命周期](filter.md#Filter的生命周期)
  * [FilterConfig](filter.md#FilterConfig)
  * [Filter的拦截方式](filter.md#Filter的拦截方式)
  * [Filter链](filter.md#Filter链)
    * [通过注解配置Filter链](filter.md#通过注解配置Filter链)
    * [通过xml配置Filter链](filter.md#通过xml配置Filter链)
  * [Filter的使用示例](filter.md#Filter的使用示例)
* [Listener监听器](listener.md)
  * [应用域监听器](listener.md#应用域监听器)
    * [ServletContextListener](listener.md#ServletContextListener)
    * [ServletContextAttributeListener](listener.md#ServletContextAttributeListener)
  * [会话域监听器](listener.md#会话域监听器)
    * [HttpSessionListener](listener.md#HttpSessionListener)
    * [HttpSessionAttributeListener](listener.md#HttpSessionAttributeListener)
    * [HttpSessionBindingListener](listener.md#HttpSessionBindingListener)
    * [HttpSessionActivationListener](listener.md#HttpSessionActivationListener)
  * [请求域监听器](listener.md#请求域监听器)
    * [ServletRequestListener](listener.md#ServletRequestListener)
    * [ServletRequestAttributeListener](listener.md#ServletRequestAttributeListener)
  * [Listener的注册方式](listener.md#Listener的注册方式)
    * [通过注解注册Listener](listener.md#通过注解注册Listener)
    * [通过xml注册Listener](listener.md#通过xml注册Listener)