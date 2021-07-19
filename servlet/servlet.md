# Servelet

## Servlet体系
![servlet+20210719092717](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719092717.png%2B2021-07-19-09-27-19)

## 开发方式
### web.xml
[官方示例](https://tomcat.apache.org/tomcat-8.5-doc/appdev/web.xml.txt)
```xml
<servlet>
    <servlet-name>servlet名称<servlet-name>
    <servlet-class>全限定类名<servlet-class>

    <!-- 更多关于servlet的配置：
        <description>
        <init-param>
            <param-name>  注释： 可有多组name-value
            <param-value>
        <load-on-startup>  注释： 取值范围1-6
     -->
<servlet>

<!-- 可以有多个servlet-mapping映射到同一个servlet -->
<servlet-mapping> 
    <servlet-name>servlet名称</servlet-name>
    <!-- 可以有多个url-pattern -->
    <url-pattern>url模式</url-pattern>

    <!-- 更多关于url映射的配置 -->
</servlet-mapping>
```

### 注解
![servlet+20210719094845](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719094845.png%2B2021-07-19-09-48-46)
```java
// 在自己编写的servlet类上进行注解

@WebServlet(
    /* 注解内容 */
)
/* Servlet类 */
```

## 映射路径
### 匹配方式
* 精确匹配：必须以“/”开头
* 模糊匹配：有且只有“*”号通配符，匹配0到多个字符，不匹配“/”
  * 前缀匹配：必须以“/”开头，以“/*”结尾
  * 后缀匹配：必须以“*”开头，以“.扩展名”结尾

### 优先级
精确 > 前缀 > 后缀

### 实现方式
* 注解：`@WebServlet(urlPatterns={"匹配路径1","匹配路径2",...})`
* web.xml：`<servlet-mapping><url-pattern>...`


## 生命周期
### 相关方法
1. init初始化方法，服务器实例Servlet之后调用，调用1次
2. service处理请求与响应的方法，浏览器请求就会调用，会被调用多次
3. destroy销毁方法，服务器关闭前销毁Servlet之前调用，调用1次

### 创建时机
浏览器默认第一次访问的时候调用了init，说明这个时候创建了Servlet对象

#### 启动创建
Servlet对象可以修改为服务器启动时创建，需要配置：
* xml方式: \<load-on-startup>1\</load-on-startup>
* 注解方式: loadOnStartup = 1

其中里面的“1”可以修改的值范围1~6，值越小，越先创建

### 创建次数
init方法值调用一次，说明servlet对象只创建一次，全局唯一对象，节省内存

### 多例模式
类实现接口SingleThreadModel后，Servlet对象是多例模式创建。但强烈不建议这么做，这会导致内存浪费。

## 运行原理
服务器根据url找到类全名，通过反射Class.forName(servlet类全名)创建了servlet对象，服务器将所有请求数据封装到request对象中，所有响应数据封装到response中，将request和response传入service方法，是使用反射调用service方法。
![servlet+20210719093753](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719093753.png%2B2021-07-19-09-37-55)


## Undetermined
### ServletConfig
Servlet配置的设置与使用
#### 设置
##### web.xml
```xml
<servlet>
    <init-param>
        <param-name>参数名key</param-name>
        <param-value>参数值value</param-value>
    </init-param>
</servlet>
```
##### 注解
```java
@WebServlet(
    initParams = {
        @WebInitParam(name = "参数名1",value = "参数值1"),
        @WebInitParam(name = "参数名2",value = "参数值1")
    }
)
```
#### 使用
```java
// GenericServlet 实例方法 getServletConfig()
ServletConfig servletConfig = genericServlet.getServletConfig()

// 获取参数
Enumeration<String> enumeration = servletConfig.getInitParameterNames();
while(enumeration.hasMoreElements()) {
    //获取当前key
    String key = enumeration.nextElement();
    // 根据key获取对应value
    String value = servletConfig.getInitParameter(key);

    doSomething(key, value);
}
```

## request
### 请求数据（行/头/体）
#### 行
* request.getMethod()：获取请求方法
* request.getRequestURI()：获取URI
* request.getRequestURL()：获取URL
* request.getProtocol()：获取协议和版本
* request.getServletPath()：获取当前Servlet的访问地址
* request.getContextPath()：获取当前项目的访问地址
* request.getRemoteAddr()：获取客户端ip地址

#### 头
* request.getHeaderNames()：得到所有的请求头名称
* request.getHeader(String headName)：获取指定请求头name对应的值value

##### 常用请求头
|请求头|描述|
|---|---|
|referer|从哪个页面跳转过来的|
|if-Modified-Since|客户端缓存静态资源的最后修改时间|
|user-Agent|描述浏览器的版本信息、浏览器内核、客户端操作系统的信息|


#### 数据
* request.setCharacterEncoding("utf-8")：设置request读取数据的字符集，避免乱码
* Enumeration\<String> request.getParameterNames()：获取所有参数名
* Map\<String,String\[]> request.getParameterMap()：获取所有参数键和值
* String request.getParameter(String name)：通过参数名获得参数值
* String[] request.getParameterValues(String name)：通过参数名获得一组同名的值（多选项目）
##### 封装
根据Map集合数据自动封装数据到JavaBean中，步骤如下：
1. 导入beanutils的jar包
2. BeanUtils.populate(Object obj, Map map);
3. *map集合中的key*与*obj中的属性字段名*要一致

### 转发
request.getRequestDispatcher("/资源路径").forward(request,response);
#### 转发与重定向的区别
重定向：response.sendRedirect("/项目路径/资源路径")

区别：
1. 转发url不变，重定向url会变。
2. 转发是在服务器内部跳转，重定向是浏览器的跳转，因此转发是1次请求，重定向是2次请求。
3. 转发跳转前后两个资源是共享request与respond，重定向不共享。

总结：需要传递数据（request、respond）就使用转发，否则使用重定向。

## 域对象
### 介绍
域对象就是服务器的内存对象，使用是有范围的。  
作用域结构是一个 Map\<String, Object>。

### 作用
实现动态资源之间传递数据

### Servlet的三大域
* request请求域
* session会话域
* servletContext全局上下文域

### API
* 写入数据，setAttribute(String key, Object value)
* 读取数据，getAttribute(String key)
* 删除数据，removeAttribute(String key)