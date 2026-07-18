# 路径结构

## 用一个例子来阐述请求路径拆分规则
示例信息：
* HTTP请求路径：http://localhost:8080/myapp/main/user/list?id=1
* 服务器地址：http://localhost:8080
* 应用上下文（ContextPath）：/myapp（部署后的项目名是myapp）
* Servlet路径（url-pattern）：/main/*
* 用户请求的具体资源：/user/list
* 查询参数：?id=1

HttpServletRequest中和路径相关的方法：

|含义|方法名|返回值示例|范畴|
|---|---|---|---|
|全路径|getRequestURL()|http://localhost:8080/myapp/main/user/list|包含协议、主机、端口和路径，但不包含查询参数。返回类型是StringBuffer。|
|资源标识符|getRequestURI()|/myapp/main/user/list|从ContextPath开始到路径结束，不含主机名和查询参数。|
|项目路径|getContextPath()|/myapp|Web应用在服务器上的根路径。如果是根部署，则返回空字符串""。|
|Servlet路径|getServletPath()|/main|匹配到该Servlet的那部分路径。|
|路径信息|getPathInfo()|/user/list|在Servlet路径之后，查询参数之前的部分。|

## URI组成的公式
`getRequestURI()` = `getContextPath()` + `getServletPath()` + `getPathInfo()`

## URL和URI的区别
* URL：Uniform Resource Locator，统一资源定位符，表示该资源在**互联网**上的绝对定位。
* URI：Uniform Resource Identifier，统一资源标识符，表示该资源在**服务器**内的标识信息。

## SpringBoot和SpringMVC中的路径结构
在SpringBoot+SpringMVC的默认配置下，整个架构和部署方式都发生了变化：
1. SpringBoot默认内嵌Tomcat，且通常一个jar包（SpringBoot打包插件产出的可运行JAR包）就只部署一个Web项目（挂载在ROOT上）；
2. SpringMVC的核心组件DispatcherServlet 默认绑定在`/`（根路径）上。

这导致了：
1. getContextPath返回空字符串
2. getServletPath返回当前访问路径（/myapp/main/user/list）
3. getPathInfo返回null
4. getRequestURI返回当前访问路径（[URI组成的公式](#uri组成的公式)依旧生效）

虽然不推荐这么做，但是SpringBoot+SpringMVC依旧支持修改上述的默认配置，使得传统的路径规则能够生效。在SpringBoot+SpringMVC项目中，推荐使用默认配置，对路径的操作使用SpringMVC提供的API来进行。