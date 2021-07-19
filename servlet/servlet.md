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