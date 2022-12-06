# Servelet
## 定义
1. Servlet 是 **JavaEE 规范**之一。规范就是接口。
2. Servlet 就 JavaWeb 三大组件之一。
3. Servlet 是运行在服务器上的一个 Java 小程序，它可以**接收客户端发送过来的请求，并响应数据给客户端**。

### 接口规范
在JavaEE平台上，处理TCP连接，解析HTTP协议这些底层工作统统扔给现成的Web服务器去做，开发者只需要把自己的应用程序跑在Web服务器上。为了实现这一目的，JavaEE提供了Servlet API，开发者使用Servlet API编写自己的Servlet来处理HTTP请求，Web服务器实现Servlet API接口，实现底层功能：
```
                 ┌───────────┐
                 │My Servlet │
                 ├───────────┤
                 │Servlet API│
┌───────┐  HTTP  ├───────────┤
│Browser│<──────>│Web Server │
└───────┘        └───────────┘
```


## Servlet体系
![servlet+20210719092717](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719092717.png%2B2021-07-19-09-27-19)

## Servlet API
Servlet API是一个jar包，需要通过Maven来引入它，才能正常编译。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>web-servlet-hello</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>5.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>hello</finalName>
    </build>
</project>
```

注意事项：
1. 打包类型不是jar，而是war，表示Java Web Application Archive
2. `<scope>`指定为`provided`，表示编译时使用，但不会打包到.war文件中，因为运行期Web服务器本身已经提供了Servlet API相关的jar包
3. Servlet版本
   1. 4.0及之前的servlet-api由Oracle官方维护，引入的依赖项是`javax.servlet:javax.servlet-api`，编写代码时引入的包名为：`import javax.servlet.*;`
   2. 5.0及以后的servlet-api由Eclipse开源社区维护，引入的依赖项是`jakarta.servlet:jakarta.servlet-api`，编写代码时引入的包名为：`import jakarta.servlet.*;`
   3. `/WEB-INF/web.xml`配置文件：低版本Servlet必须有，高版本Servlet不需要

## 开发方式
我们需要开发自己的Servlet程序，并将Servlet程序注册/告诉到tomcat，让tomcat使用我们开发的Servlet程序处理请求并响应数据。

有两种方式开发方式：
* 基于配置：在web.xml中配置自己的编写好的Servlet程序，以供tomcat读取
* 基于注解(推荐)：在Servlet上用注解修饰，以供tomcat扫描
### web.xml
[官方示例](https://tomcat.apache.org/tomcat-8.5-doc/appdev/web.xml.txt)
```xml
<!-- servlet 标签给 Tomcat 配置 Servlet 程序 -->
<servlet>  
  <!--servlet-name 标签 Servlet 程序起一个别名(一般是类名) -->
  <servlet-name>servlet名称<servlet-name>
  <!--servlet-class 是 Servlet 程序的全类名-->
  <servlet-class>全限定类名<servlet-class>

  <!-- 更多关于servlet的配置：
    <description>
    <init-param>
      <param-name>  注释： 可有多组name-value
      <param-value>
    <load-on-startup>  注释： 取值范围1-6
  -->
<servlet>

<!--servlet-mapping 标签给 servlet 程序配置访问地址-->
<!-- 可以有多个servlet-mapping映射到同一个servlet -->
<servlet-mapping> 
  <!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个 Servlet 程序使用-->
  <servlet-name>servlet名称</servlet-name>
  <!--url-pattern 标签配置访问地址-->
  <!-- 可以有多个url-pattern -->
  <url-pattern>url模式</url-pattern>

  <!-- 更多关于url映射的配置 -->
</servlet-mapping>
```

### 注解
```java
// 在自己编写的servlet类上进行注解

@WebServlet(
    /* 注解内容 */
)
/* Servlet类 */
```

在@WebServlet中可以配置如下可选参数：
![servlet+20210719094845](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719094845.png%2B2021-07-19-09-48-46)


## 映射路径
### 匹配方式
* 精确匹配：必须以“/”开头
* 模糊匹配：有且只有“*”号通配符，匹配0到多个字符，不匹配“/”
  * 前缀匹配：必须以“/”开头，以“/*”结尾
  * 后缀匹配：必须以“*”开头，以“.扩展名”结尾

### 优先级
精确 > 前缀 > 后缀

### 实现方式
* web.xml：`<servlet-mapping><url-pattern>...`
* 注解：`@WebServlet(urlPatterns={"匹配路径1","匹配路径2",...})`


## 生命周期

### 相关方法
0. 构造器方法
1. init初始化方法，服务器实例Servlet之后调用，调用1次
2. service处理请求与响应的方法，浏览器请求就会调用，会被调用多次
3. destroy销毁方法，服务器关闭前销毁Servlet之前调用，调用1次

### 创建时机

#### 第一次访问时创建(默认情况)
浏览器默认第一次访问的时候调用了init，说明这个时候创建了Servlet对象

#### 服务器启动时创建(可选配置)
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


## ServletConfig类
ServletConfig 类是 Servlet 程序的配置信息类；由 Tomcat 负责创建，程序员负责使用；每个 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象。
### 作用
1. 可以获取 Servlet 程序的别名 servlet-name 的值
2. 获取初始化参数 init-param
3. 获取 ServletContext 对象
### 设置
#### web.xml
```xml
<servlet>
    <init-param>
        <param-name>参数名key</param-name>
        <param-value>参数值value</param-value>
    </init-param>
</servlet>
```
#### 注解
```java
@WebServlet(
    initParams = {
        @WebInitParam(name = "参数名1",value = "参数值1"),
        @WebInitParam(name = "参数名2",value = "参数值1")
    }
)
```
### 使用
```java
// GenericServlet 实例方法 getServletConfig()
ServletConfig servletConfig = genericServlet.getServletConfig()

// 获取 Servlet 程序的别名 servlet-name 的值
System.out.println("程序的别名是:" + servletConfig.getServletName());

// 获取初始化参数 init-param
Enumeration<String> enumeration = servletConfig.getInitParameterNames();
while(enumeration.hasMoreElements()) {
    //获取当前key
    String key = enumeration.nextElement();
    // 根据key获取对应value
    String value = servletConfig.getInitParameter(key);

    doSomething(key, value);
}

// 获取 ServletContext 对象
System.out.println(servletConfig.getServletContext());
```


## request
```java
HttpServletRequest request;
```
每次只要有请求进入 Tomcat 服务器，Tomcat 服务器就会把请求过来的 HTTP 协议信息解析好封装到 Request 对象中，然后传递到 service 方法(doGet 和 doPost)中给我们使用。我们可以通过 HttpServletRequest 对象，获取到所有请求的信息。
### 请求数据（行/头/体）
#### 行
* request.getMethod()：获取请求方法
* request.getRequestURI()：获取请求的资源路径，不包括请求参数
* request.getQueryString()：返回请求参数
* request.getParameter(name)：返回请求参数，GET请求从URL读取参数，POST请求从Body中读取参数
* request.getRequestURL()：获取请求的统一资源定位符(绝对路径)
* request.getRemoteAddr()：返回客户端的IP地址
* request.getScheme()：返回协议类型，例如"http"，"https"
* request.getProtocol()：获取协议和版本
* request.getServletPath()：获取当前Servlet的访问地址
* request.getContextPath()：获取当前项目的访问地址

#### 头
* request.getHeaderNames()：得到所有的请求头名称
* request.getHeader(String headName)：获取指定请求头name对应的值value
* request.getCookies()：返回请求携带的所有Cookie
##### 常用请求头
| 请求头            | 描述                                                   |
| ----------------- | ------------------------------------------------------ |
| referer           | 从哪个页面跳转过来的                                   |
| if-Modified-Since | 客户端缓存静态资源的最后修改时间                       |
| user-Agent        | 描述浏览器的版本信息、浏览器内核、客户端操作系统的信息 |

#### 数据
* request.setCharacterEncoding("utf-8")：设置request读取数据的字符集，避免乱码
* Enumeration\<String> request.getParameterNames()：获取所有参数名
* Map\<String,String\[]> request.getParameterMap()：获取所有参数键和值
* String request.getParameter(String name)：通过参数名获得参数值（参数值为单值/一个字符串）
* String[] request.getParameterValues(String name)：通过参数名获得一组同名的值（参数值为一个字符串数组）
* request.getInputStream()：如果该请求带有HTTP Body，该方法将打开一个输入流用于读取Body
* request.getReader()：和getInputStream()类似，但打开的是Reader
* request.getContentType()：获取请求Body的类型，例如"application/x-www-form-urlencoded"
##### 封装
根据Map集合数据自动封装数据到JavaBean中，步骤如下：
1. 导入beanutils的jar包
2. BeanUtils.populate(Object obj, Map map);
3. *map集合中的key*与*obj中的属性字段名*要一致

### 转发
转发：request.getRequestDispatcher("/资源路径").forward(request,response);
#### 转发与重定向
重定向API：response.sendRedirect("/项目路径/资源路径")

项目路径：`request.getContextPath()`获取当前Webapp挂载的路径，对于ROOT来说，总是返回空字符串""。

重定向过程：
```
┌───────┐   GET /hi     ┌───────────────┐
│Browser│ ────────────> │RedirectServlet│
│       │ <──────────── │               │
└───────┘   302 /hello  └───────────────┘


┌───────┐   GET /hello  ┌───────────────┐
│Browser│ ────────────> │ HelloServlet  │
│       │ <──────────── │               │
└───────┘   200 <html>  └───────────────┘
```

#### 区别
1. 转发url不变，重定向url会变。
2. 转发是在服务器内部跳转，重定向是浏览器的跳转，因此转发是1次请求，重定向是2次请求。
3. 转发跳转前后两个资源是共享request与response，重定向不共享。

#### 总结
需要传递数据（request、response）就使用转发，否则使用重定向。


## response
```java
HttpServletResponse response;
```
每次请求进来，Tomcat 服务器都会创建一个 Response 对象传递给 Servlet 程序去使用。我们如果需要设置返回给客户端的信息，都可以通过 HttpServletResponse 对象来进行设置。

### 响应数据（行/头/体）
#### 行
* response.setStatus(int status)：设置状态码
* response.sendError(int sc, String msg)：发送一个错误码和错误信息

#### 头
* response.setHeader(String name, String value)：用给定名称和值设置响应头
* response.addHeader(name, value)：给响应添加一个Header，因为HTTP协议允许有多个相同的Header
* response.setContentType(String type)：设置Body的类型，例如，"text/html"
* response.setCharacterEncoding(String charset)：设置字符编码，例如，"UTF-8"
* response.addCookie(Cookie cookie)：给响应添加一个Cookie

##### 常见响应头
| 响应头              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| location            | 重定向跳转的地址                                             |
| content-length      | 响应体的数据长度                                             |
| content-type        | 设置响应类型;设置浏览器的码表。例如：text/html;charset=utf-8 |
| refresh             | 多少秒后跳转;url=重定向跳转的地址                            |
| content-disposition | 内容的处理方式，可以控制下载。attachment;filename=xxx.xx     |
| content-encoding    | 压缩方式。gzip                                               |

#### 体
* 获得响应体的输出流，输出数据（二者只能获取其中一个）
  * var writer = response.getWriter()：字符流输出
    * writer.print(任何类型)：本质将任何类型转换为字符串输出，兼容性更好
    * writer.write(字符串类型)
  * response.getOutputStream()：字节流输出
* 设置响应体的类型，避免乱码
  * response.setContentType("text/html;charset=utf-8")
  * 上面的一句(推荐) 等价于 下面的两句
  * response.setCharacterEncoding("utf-8");
  * response.setHeader("content-type","text/html;charset=utf-8");
* 数据压缩
  * GZIPOutputStream(OutputStream out)：使用默认缓冲区大小创建新的输出流（用于包装response.getOutputStream()）。
  * public void write(byte[] b) 将字节数组写入压缩输出流。
  * void finish() 完成将压缩数据写入输出流的操作，无需关闭底层流。
* 重定向
  * response.sendRedirect("/项目路径/资源路径")
  * 上面的一句(推荐) 等价于 下面的两句
  * response.setStatus(302);
  * response.setHeader("Location", "/项目路径/资源路径");

![servlet+1565982684673](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B1565982684673.png%2B2021-07-19-15-01-53)
[Url编码-百分号编码介绍](../front-end/url-encode.md)  
[base64编码](../front-end/base64.md)

### 注意事项
1. 写入响应前，无需设置setContentLength()，因为底层服务器会根据写入的字节数自动设置，如果写入的数据量很小，实际上会先写入缓冲区，如果写入的数据量很大，服务器会自动采用Chunked编码让浏览器能识别数据结束符而不需要设置Content-Length头。
2. 写入完毕后，必须调用flush()，因为大部分Web服务器都基于HTTP/1.1协议，会复用TCP连接。如果没有调用flush()，将导致缓冲区的内容无法及时发送到客户端。此外，写入完毕后千万不要调用close()，原因同样是因为会复用TCP连接，如果关闭写入流，将关闭TCP连接，使得Web服务器无法复用此TCP连接。


## 域对象
### 介绍
域对象就是服务器的内存对象，使用是有范围的。  
作用域结构是一个 Map\<String, Object>。

### 作用
实现动态资源之间传递数据

### Servlet的三大域
| 作用域对象                 | 作用域范围     | 应用场景                                                         |
| -------------------------- | -------------- | ---------------------------------------------------------------- |
| request请求域              | 一次请求内     | 请求转发跳转页面传递数据                                         |
| session会话域              | 一个会话内     | 存储验证码，存储用户登录数据                                     |
| servletcontext全局上下文域 | 整个应用程序内 | 统计全局的数据，这个数据全局共享，所有用户共享，例如统计登录人数 |

### 通用API
* 写入数据，setAttribute(String key, Object value)
* 获取所有属性名，getAttributeNames()
* 读取数据，getAttribute(String key)
* 删除数据，removeAttribute(String key)

### servletContext全局上下文域

#### 定义
1. ServletContext 是一个接口，它表示 Servlet 上下文对象
2. 一个 web 工程，只有一个 ServletContext 对象实例
3. ServletContext 在 web 工程部署启动的时候创建，在 web 工程停止的时候销毁

#### 作用
1. 获取 web.xml 中配置的上下文参数 context-param
2. 获取当前的工程路径
3. 获取工程部署后在服务器硬盘上的绝对路径（并获取资源）
4. 像 Map 一样存取数据

#### 获取方式
![servlet+20210719165959](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719165959.png%2B2021-07-19-17-00-00)
```java
ServletContext servletcontext = getServletContext();
```

#### 读取资源文件
* String servletcontext.getContextPath()：获取当前的工程路径
* String servletcontext.getRealPath(相对路径)：根据相对路径获取项目部署位置上资源的绝对路径
* InputStream servletcontext.getResourceAsStream(相对路径)：根据项目资源相对路径获取部署资源文件的输入流

#### 读取全局配置参数
全局配置参数是在web.xml中\<context-param>配置的参数，示例如下：
```xml
<!--context-param 是上下文参数(它属于整个 web 工程)--> 
<context-param>
  <param-name>username</param-name>
  <param-value>root</param-value>
</context-param>
<!--context-param 是上下文参数(它属于整个 web 工程)--> 
<context-param>
  <param-name>password</param-name>
  <param-value>root</param-value>
</context-param>
```

* Enumeration\<String> servletcontext.getInitParameterNames() 得到所有的全局参数名
* String servletcontext.getInitParameter(String name) 指定参数的名字，得到值

### session会话域
cookie是用于在客户端浏览器存储会话数据。Cookie 属于客户端会话技术，数据保存在浏览器端文件中，Cookie 中键和值都是 String 类型

Session 属于服务器端的会话技术，数据保存服务器内存中，Session 中键是 String，值是 Object 类型。服务器会为每个浏览器（每个用户）都会创建独享的session对象。

#### session和cookie
通常session技术是依赖cookie技术（Set-Cookie: JSESSIONID=会话ID），但还有[其他方式](https://www.zhihu.com/question/35307626)可选。

![servlet+20210722165512](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210722165512.png%2B2021-07-22-16-55-16)

##### 对比
| 特点                 | cookie   | session                        |
| -------------------- | -------- | ------------------------------ |
| 数据存储在哪一端     | 客户端   | 服务器端                       |
| 数据大小是否有限制   | 4kb      | 没有                           |
| 存储存储什么类型数据 | string   | object                         |
| 存储的数据默认有效期 | 会话结束 | 会话结束或者是非活动时间30分钟 |
| 存储数据安全吗？     | 不安全   | 安全                           |

#### Cookie相关的API
##### Cookie
* Cookie(String name,String value)：构造方法，创建一个Cookie对象
* String getName()：得到cookie的键
* String getValue()：得到cookie的值
* setMaxAge(int expiry)：设置cookie过期时间，单位是秒。正数：设置秒数；负数：浏览器关闭就失效；零：删除cookie。
* setPath(String uri)：设置cookie的访问路径，只有访问这个路径或者它的子路径，浏览器才会将cookie发送给服务器。

##### 写/读
* response.addCookie(Cookie cookie)：将服务器创建的cookie通过响应发送给浏览器
* Cookie[] request.getCookies()：服务器得到浏览器发送过来的所有的cookie信息，返回的是一个Cookie的对象数组。如果没有读到任何的Cookie，返回null

##### 编码
在 cookie 值中不能使用分号（;）、逗号（,）、等号（=）以及空格，Tomcat8 中可以直接使用汉字，Tomcat7 中 Cookie 的值不能使用汉字。

如果要使用特殊字符，必须在写入字符值之前进行编码。读取以后再进行解码:
* URLEncoder.encode("字符串", "utf-8")：把字符串使用utf8进行编码
* URLDncoder.dncode("字符串", "utf-8")：把字符串使用utf8进行解码

#### Session相关的API
##### HttpSession
* HttpSession request.getSession()：通过请求对象获得会话对象，不存在就创建，存在就返回。
* String getId()：得到会话ID，在服务器上唯一的32位的十六进制数。作用：“Set-Cookie: JSESSIONID=会话ID”。
* long getCreationTime()：会话创建的时间，1970-1-1到这个时间的之间相差的毫秒数
* boolean isNew()：判断是否位新的会话，是的返回ture
* int getMaxInactiveInterval()：得到服务器上会话最大的非活动时间间隔，默认是1800秒（30分钟）
* setMaxInactiveInterval(int interval)：设置会话最大活动时间间隔，单位是秒
* invalidate()：让会话立刻失效，一般用于用户退出注销

##### 有效期
session在服务器内存中不是永久的，默认距离上一次请求间隔超过30分钟会被销毁。

可以在web.xml统一修改所有session过期时间：
```xml
<session-config>
    <!-- 单位是分钟 -->
    <session-timeout>30</session-timeout>
</session-config>
```

#### 修改JSESSIONID有效期（客户端）
浏览器关闭时，默认生成的cookie数据JSESSIONID过期了，导致打开浏览器第二次访问服务器不会携带JSESSIONID。只要延长JSESSIONID过期时间，服务器就会获取其对应的旧session返回使用。
```java
//1.创建Session对象
HttpSession session = request.getSession();

//2.获取sessionid
String sessionId = session.getId();

//3.创建cookie写入JSESSIONID数据并设置过期时间更长
Cookie cookie = new Cookie("JSESSIONID", sessionId);
cookie.setMaxAge(60*60*24); //1天

//4.输出给浏览器去覆盖默认的cookie数据
response.addCookie(cookie);
```

#### 钝化与激活
##### 钝化
服务器正常关闭，会将内存session数据持久化到服务器磁盘上，类似于序列化对象过程

序列化磁盘位置：$CATALINA_HOME\work\Catalina\localhost\项目名\SESSIONS.ser
##### 激活
服务器启动时，会将磁盘数据恢复到内存会话域对象中，类似于反序列化对象过程
##### 注意
1. idea无法演示钝化与激活，必须将项目部署到正式tomcat上，单独启动与关闭服务器测试才有效。因为idea部署的机制先删除所有数据再进行部署，这个删除会将序列化文件也删除了。
2. 由于进行序列化与反序列操作，需要会话的中的对象数据所属的类实现可序列化接口

#### 禁用Cookie的解决方案
重定向的url重写，把会话的ID在地址栏上带给服务器：`response.encodeURL(path)`。

```java
1.创建session
HttpSession session = request.getSession();

//2.写入数据
session.setAttribute("username","admin");

//使用url重写解决
String url = response.encodeURL(request.getContextPath() + "/Servlet2");
/* 
实现原理：
就是在这个路径request.getContextPath() + "/Servlet2" 后面拼接 “;jsessionid=sessionid”

最终得到：
xxx/Servlet2;jsessionid=4C927EA8EBEBBCE600B6444772BF7951
 */

//3.跳转到Servlet2去读取展现
response.sendRedirect(url);
```