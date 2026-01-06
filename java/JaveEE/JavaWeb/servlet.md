# Servelet
在JavaEE平台上，处理TCP连接、解析HTTP协议这些底层工作统统扔给现成的Web服务器去做，开发者只需要把自己的应用程序跑在Web服务器上。为了实现这一目的，JavaEE提供了Servlet API，开发者使用Servlet API编写自己的Servlet来处理HTTP请求，Web服务器实现Servlet API接口，实现底层功能。因此Servlet就是一个运行在JavaWeb服务器中的一个程序，它接收客户端发送过来的请求，执行相应的处理逻辑，并将响应的数据返回给客户端。



## Servlet的继承体系
![servlet+20210719092717](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719092717.png%2B2021-07-19-09-27-19)



## Servlet的开发示例
[使用IDEA开发Servlet开发的示例](servlet_develop_demo.md)



## Servlet的开发流程
1. 首先我们需要开发自己的Servlet程序（编写相关代码）
2. 然后将Servlet程序注册到Tomcat，有两种方式注册方式：
   1. 在WEB-INF/web.xml中配置
   2. 在Servlet上使用@WebServlet注解进行修饰
3. 最后Tomcat会使用我们开发的Servlet程序处理请求并响应数据



## Servlet的两种注册方式

#### 通过xml注册Servlet
[官方示例](https://tomcat.apache.org/tomcat-8.5-doc/appdev/web.xml.txt)
```xml
<!-- servlet 标签给 Tomcat 配置 Servlet 程序 -->
<servlet>  
    <!-- servlet-name标签Servlet程序起一个别名（一般是类名，小驼峰） -->
    <servlet-name>servlet名称<servlet-name>
    <!-- servlet-class是Servlet程序的全类名-->
    <servlet-class>全限定类名<servlet-class>

    <!-- 更多关于servlet的配置：
        <description>
        <init-param>
        <param-name> 注释：可有多组name-value
        <param-value>
        <load-on-startup> 注释：取值范围1-6
    -->
<servlet>

<!-- servlet-mapping标签给Servlet程序配置访问地址-->
<!-- 可以有多个servlet-mapping映射到同一个Servlet -->
<servlet-mapping> 
    <!-- servlet-name标签的指示当前配置的地址给指向哪个Servlet程序 -->
    <servlet-name>servlet名称</servlet-name>
    <!-- url-pattern标签配置访问地址 -->
    <!-- 可以有多个url-pattern -->
    <url-pattern>url模式</url-pattern>

    <!-- 更多关于url映射的配置 -->
</servlet-mapping>
```

#### 通过注解注册Servlet
```java
// 在自己编写的servlet类上进行注解
@WebServlet( ... )
public class MyServlet extends HttpServlet { ... }
```

在@WebServlet中可以配置如下可选参数：

![servlet+20210719094845](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719094845.png%2B2021-07-19-09-48-46)



## Servlet的映射规则
Servlet的映射规则可以在web.xml或@WebServlet注解中进行配置：
* web.xml：  
  ```xml
  <servlet-mapping>
      <servlet-name>servlet名称</servlet-name>
      <url-pattern>url模式</url-pattern>
  </servlet-mapping>
  ```
* @WebServlet注解：  
  ```Java
  @WebServlet(urlPatterns = "url模式")
  public class MyServlet extends HttpServlet { ... }
  ```

### url的匹配模式
在url匹配中，有且只有星号匹配符这一种匹配符，不存在其他的匹配符。星号匹配符可以匹配0到多个字符，但是不能匹配正斜杠（<b>/</b>）字符。

在进行url匹配时，有如下形式：
* 匹配所有：`/*`能够匹配所有请求。
* 拓展名匹配：`*.ext`，例如`*.jsp`能够匹配所有jsp文件请求。
* 目录匹配：`/path/*`，例如`/users/*`能够匹配`/users/`、`/users/profile`、`/users/details`等。
* 精确匹配：例如`/exactPath`只能匹配`/exactPath`。

### 匹配的优先级
一个url路径满足多个映射条件时，遵循以下优先级：
1. 精确匹配：例如，如果同时有`/login`和`/login/*`，请求`/login`会优先匹配精确的`/login`。
2. 最长前缀匹配：如果存在`/app/user/*`和`/app/user/profile/*`，那么`/app/user/profile/info`会匹配后者。
3. 目录匹配：`/app/*`优于`/*`。
4. 拓展名匹配：优先级低于路径匹配。
5. `/*`：最后的兜底。

### 常见写法
* `/`：表示通配所有资源,不包括jsp文件
* `/*`：表示通配所有资源,包括jsp文件
* `/a/*`：匹配所有以a前缀的映射路径
* `*.action`：匹配所有以action为后缀的映射路径



## Servlet的生命周期

### 相关方法
0. 构造器方法，创建Java对象时被自动调用，会被调用1次。
1. init初始化方法，实例化Servlet之后被调用，会被调用1次。
2. service方法，处理请求与返回响应时会被调用的方法，每次收到请求就会被调用，会被调用多次。
3. destroy销毁方法，在Web容器关闭的时候，销毁Servlet之前被调用，会被调用1次。

### 创建时机

#### 第一次访问时创建(默认情况)
浏览器默认第一次访问的时候调用了init，说明这个时候创建了Servlet对象。

#### 服务器启动时创建(可选配置)
Servlet对象可以修改为服务器启动时创建，需要配置：
* xml方式：&lt;load-on-startup>1&lt;/load-on-startup>
* 注解方式：loadOnStartup = 1

其中里面的“1”可以修改的值范围1~6，值越小，越先创建

### 创建次数
Servlet对象只创建一次，全局唯一对象，节省内存。

### 多例模式
类实现接口SingleThreadModel后，Servlet对象是多例模式创建。但强烈不建议这么做，这会导致内存浪费。



## Servlet的运行原理
服务器根据url找到类全名，通过反射`Class.forName(servlet全类名)`创建了Servlet对象，服务器将所有请求数据封装到HttpServletRequest对象中，所有响应数据封装到HttpServletResponse对象中，将HttpServletRequest对象和HttpServletResponse对象传入service方法，使用反射调用service方法。

![servlet+20210719093753](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719093753.png%2B2021-07-19-09-37-55)



## ServletConfig
ServletConfig是Servlet的配置信息类。在每个Servlet创建时，Web容器也会创建一个对应的ServletConfig对象，并通过Servlet生命周期的init方法传入给Servlet作为其属性。

```Java
public interface ServletConfig {
    // 获取Servlet的名称
    String getServletName();

    // 获取ServletContext对象
    ServletContext getServletContext();

    // 根据名字，获取配置Servlet时设置的初始化参数
    String getInitParameter(String var1);

    // 获取所有初始化参数名组成的Enumeration对象
    Enumeration<String> getInitParameterNames();
}
```

### ServletConfig的作用
1. 获取Servlet的别名*servlet-name*的值。
2. 获取Servlet的初始化参数*init-param*。
3. 获取*ServletContext*对象。

### 设置ServletConfig两种方式

#### 在xml中设置ServletConfig
```xml
<servlet>
    <init-param>
        <param-name>参数名key</param-name>
        <param-value>参数值value</param-value>
    </init-param>
</servlet>
```
#### 在注解中设置ServletConfig
```Java
@WebServlet(
    initParams = {
        @WebInitParam(name = "参数名1",value = "参数值1"),
        @WebInitParam(name = "参数名2",value = "参数值1")
    }
)
public class MyServlet{ ... }
```

### 在Servlet中使用ServletConfig
```Java
// GenericServlet的实例方法getServletConfig()
ServletConfig servletConfig = genericServlet.getServletConfig();

// 获取Servlet程序的别名servlet-name的值
System.out.println("程序的别名是：" + servletConfig.getServletName());

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


## HttpServletRequest
```java
HttpServletRequest req;
```
每次只要有请求进入Tomcat服务器，Tomcat服务器就会把请求过来的HTTP协议信息解析好封装到HttpServletRequest对象中，然后传递到service方法中给我们使用。我们可以通过HttpServletRequest对象，获取到所有请求的信息。

### 请求数据
#### 请求行
* String getMethod()：获取请求方法。
* StringBuffer getRequestURL()：获取请求的统一资源定位符（形如`协议://主机:端口/路径`，该*路径*包含*Web项目名*，不包括查询字符串）。
* String getRequestURI()：获取请求的资源路径（形如`/路径`，该*路径*包含*Web项目名*，不包括查询字符串）。
* String getQueryString()：返回查询字符串。
* String getParameter(String name)：返回请求参数；GET请求从URL读取参数，POST请求从Body中读取参数；如果该参数有多个值，则该方法返回第一个值。
* String[] getParameterValues(String name)：返回请求参数，该方法返回的是一个字符串数组。
* String getRemoteAddr()：返回客户端的IP地址。
* String getScheme()：返回协议类型，例如"http"或"https"。
* String getProtocol()：获取协议和版本，例如"HTTP/1.1"
* String getServletPath()：获取当前Servlet的访问路径，与@WebServlet注解中urlPatterns元素相关：
  * 当urlPatterns="/*"时，返回""；
  * 当urlPatterns="*.ext"时，返回".ext"；
  * 当urlPatterns="/servletA/*"时，返回"/servletA"；
  * 当urlPatterns="/servletB"时，返回"/servletB"。
* String getContextPath()：获取当前Web项目的上下文路径，指的是[部署时Web项目文件夹的名称](tomcat.md#webapps真实目录发布)或者是[使用配置server.xml进行部署时设定的Web项目路径名](tomcat.md#虚拟目录发布)。

#### 请求头
* Enumeration&lt;String> getHeaderNames()：得到所有的请求头名称。
* String getHeader(String name)：获取指定请求头name对应的值value。
* Cookie[] getCookies()：返回请求携带的所有Cookie。

#### 请求体
* void setCharacterEncoding(String env)：设置从请求体中读取数据时使用的字符集，避免乱码。
* Enumeration&lt;String> req.getParameterNames()：获取所有参数名。
* Map&lt;String, String[]> getParameterMap()：获取所有参数键和值。
* String req.getParameter(String name)：通过参数名获得参数值（参数值为一个字符串）。
* String[] req.getParameterValues(String name)：通过参数名获得一组同名的值（参数值为一个字符串数组）。
* ServletInputStream getInputStream()：如果该请求带有HTTP Body，该方法将打开一个输入流用于读取Body。
* BufferedReader getReader()：和getInputStream()类似，但打开的是Reader。
* String getContentType()：获取请求Body的类型，例如"application/x-www-form-urlencoded"。



## HttpServletResponse
```java
HttpServletResponse resp;
```
每次请求进来，Tomcat服务器都会创建一个HttpServletResponse对象传递给Servlet程序去使用。我们如果需要设置返回给客户端的信息，可以通过HttpServletResponse对象进行设置。

### 响应数据
#### 响应行
* void setStatus(int sc)：设置状态码。
* void sendError(int sc, String msg)：设置错误码和错误信息。

#### 响应头
* void setHeader(String name, String value)：用给定名称和值设置响应头。
* void addHeader(String name, String value)：给响应添加一个Header，因为HTTP协议允许一个响应头有多个值。
* void setContentType(String type)：设置Body的类型，例如："text/html"。
* void setCharacterEncoding(String charset)：设置字符编码，例如："UTF-8"。
* void addCookie(Cookie cookie)：给响应添加一个Cookie。

##### 常见响应头
|响应头|描述|
|---|---|
|location|重定向跳转的地|
|content-length|响应体的数据长|
|content-type|设置响应类型。设置字符编码。例如：text/html;charset=utf-8|
|refresh|多少秒后跳转。配合"url=重定向跳转的地址"使用|
|content-disposition|内容的处理方式，可以控制浏览器执行下载操作，形如：attachment;filename=xxx.xx|
|content-encoding|压缩方式。例如：gzip|

#### 响应体
Servlet API中提供的操作响应体*resp*的方法，主要用于实现如下的功能：
* 获取响应体的输出流后，进行数据输出的操作，只能从字符流和字节流的二者中择一进行操作：
  * `var writer = resp.getWriter();`：获取字符流输出
    * `writer.print(任何类型);`：该方法的本质是将任何类型的对象通过String.valueOf静态方法转换为字符串后进行输出，因此有较好的兼容性
    * `writer.write(字符串类型);`
  * `resp.getOutputStream();`：获取字节流输出
* 设置响应体的类型，避免乱码
  * `resp.setContentType("text/html;charset=utf-8");`
  * 上面的一句(推荐) 等价于 下面的两句
  1. `resp.setCharacterEncoding("utf-8");`
  2. `resp.setHeader("content-type","text/html;charset=utf-8");`
* 数据压缩
  1. `var gzos = new GZIPOutputStream(OutputStream out);`：使用默认缓冲区大小创建新的输出流（用于包装resp.getOutputStream()返回的字节输出流）。
  2. `gzos.write(字节数组);`：将字节数组写入压缩输出流。
  3. `gzos.finish();`：完成将压缩数据写入输出流的操作后，无需关闭底层流，调用该方法即可。
* 重定向
  * `resp.sendRedirect("/项目路径/资源路径");`：返回302表示临时重定向
  * 上面的一句(推荐) 等价于 下面的两句
  1. `resp.setStatus(302);`，也可以传入301表示永久重定向
  2. `resp.setHeader("Location", "/项目路径/资源路径");`

![servlet+1565982684673](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B1565982684673.png%2B2021-07-19-15-01-53)
[Url编码-百分号编码介绍](../front-end/url-encode.md)  
[base64编码](../front-end/base64.md)

写入响应体前后的注意事项：
1. 写入响应前，无需设置setContentLength()，因为底层服务器会根据写入的字节数自动设置，如果写入的数据量很小，实际上会先写入缓冲区，如果写入的数据量很大，服务器会自动采用Chunked编码让浏览器能识别数据结束符而不需要设置Content-Length头。
2. 写入完毕后，必须调用flush()，因为大部分Web服务器都基于HTTP/1.1协议，会复用TCP连接。如果没有调用flush()，将导致缓冲区的内容无法及时发送到客户端。此外，写入完毕后千万不要调用close()，原因同样是因为会复用TCP连接，如果关闭写入流，将关闭TCP连接，使得Web服务器无法复用此TCP连接。


## 转发和重定向
**转发**和**重定向**是*间接访问*项目资源的两种手段，也是Servlet控制页面跳转的两种手段。

|异同点|**转发**|**重定向**|
|---|---|---|
|实现方式|通过HttpServletRequest对象实现|通过HttpServletResponse对象实现|
|客户端是否感知|转发是服务器内部的行为，对客户端是屏蔽的|重定向是服务端将<i>响应码（301或302等）</i>和<i><u>新的访问路径</u></i>告知客户端，随后让客户端主动向<i><u>新的访问路径</u></i>发起请求|
|请求次数|客户端只发送了一次请求|客户端至少发送了两次请求|
|地址栏是否改变|否|是|
|请求对象和响应对象是否会传递给下一个资源|是|否|
|请求参数是否可以传递|是|否|
|请求域数据是否可以传递|是|否|
|是否可以访问WEB-INF下的资源|是|否|
|是否可以访问本项目外的资源|否|是|

### 转发和重定向的代码示例
![f5cc5da1-8344-46fb-aae9-23ad34bcce5d](https://raw.githubusercontent.com/loli0con/picgo/master/f5cc5da1-8344-46fb-aae9-23ad34bcce5d.png)

#### 转发的代码示例
```Java
@WebServlet("/servletA")
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取转发器 req.getRequestDispatcher("资源路径")

        // 转发给servlet
        RequestDispatcher requestDispatcher = req.getRequestDispatcher("servletB");

        // 转发给一个视图资源 OK
        // RequestDispatcher requestDispatcher = req.getRequestDispatcher("welcome.html");

        // 转发给WEB-INF下的资源 OK
        // RequestDispatcher requestDispatcher = req.getRequestDispatcher("WEB-INF/views/view1.html");

        //  转发给外部资源 NO
        // RequestDispatcher requestDispatcher = req.getRequestDispatcher("http://www.google.com");

        //  获取请求参数
        String username = req.getParameter("username");
        System.out.println(username);

        // 向请求域中添加数据
        req.setAttribute("reqKey", "requestMessage");

        // 执行转发动作
        requestDispatcher.forward(req, resp);
    }
}



@WebServlet("/servletB")
public class ServletB extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取请求参数（转发过程可以获取到请求参数）
        String username = req.getParameter("username");
        System.out.println(username);

        // 获取请求域中的数据（转发过程可以获取到请求域中的数据）
        String reqMessage = (String) req.getAttribute("reqKey");
        System.out.println(reqMessage);

        // 做出响应
        resp.getWriter().write("servletB response");
    }
}
```

#### 重定向的代码示例
```Java
@WebServlet("/servletA")
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws IOException, IOException {
        // 获取请求参数
        String username = req.getParameter("username");
        System.out.println(username);

        // 向请求域中添加数据
        req.setAttribute("reqKey", "requestMessage");

        // 响应重定向
        // 重定向到servlet动态资源 OK
        resp.sendRedirect("servletB");

        // 重定向到视图静态资源 OK
        // resp.sendRedirect("welcome.html");

        // 重定向到WEB-INF下的资源 NO
        // resp.sendRedirect("WEB-INF/views/view1");

        // 重定向到外部资源 OK
        // resp.sendRedirect("http://www.google.com");
    }
}

@WebServlet("/servletB")
public class ServletB extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取请求参数（重定向过程将丢失请求参数）
        String username = req.getParameter("username");
        System.out.println(username);

        // 获取请求域中的数据（重定向过程将丢失请求域中的数据）
        String reqMessage = (String) req.getAttribute("reqKey");
        System.out.println(reqMessage);

        // 做出响应
        resp.getWriter().write("servletB response");
    }
}
```


## 三大作用域对象
域对象是一些用于存储数据和传递数据的对象。根据传递数据的范围的不同，我们将其划分为不同的域。不同的域对象代表不同的域，共享数据的范围也不同。

|作用域对象|作用域范围|应用场景|
|---|---|---|
|HttpServletRequest请求域|一次请求内|**转发**跳转页面传递数据|
|HttpSession会话域|一个会话内|存储验证码、用户登录数据等|
|Servletcontext全局上下文域|整个Web应用程序内|统计全局的数据。这个数据全局共享，所有用户共享，可用于统计登录人数等|

### 通用API
* 根据属性名写入数据：`setAttribute(String key, Object value)`
* 获取所有属性名：`getAttributeNames()`
* 根据属性名读取数据：`getAttribute(String key)`
* 根据属性名删除数据：`removeAttribute(String key)`

### ServletContext全局上下文域
1. ServletContext是一个接口，它表示Servlet上下文对象
2. Web容器会为每一个Web项目创建一个对应的ServletContext对象
3. ServletContext在Web项目启动的时候创建，在Web项目停止的时候销毁
4. ServletContext对象被所有的Web组件所共享，可以给所有的Web组件提供参数配置功能。

#### ServletContext的作用
1. 获取web.xml中配置的上下文参数
   ```xml
   <context-param>
       <param-name>参数名</param-name>
       <param-value>参数值</param-value>
   </context-param>
   ```
2. 获取当前的Web项目的路径
3. 获取当前Web项目部署后在服务器硬盘上的绝对路径（并获取服务器上的资源Resource）
4. 像Map一样存取参数数据

#### 获取ServletContext的方法
![servlet+20210719165959](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210719165959.png%2B2021-07-19-17-00-00)
```java
ServletContext servletcontext = getServletContext();
```

#### 初始参数读取
```Java
// 获取特定的参数值
ServletContext servletContext = this.getServletContext();
String valueA = servletContext.getInitParameter("paramA");
System.out.println("paramA: " + valueA);

// 获取所有的参数名
Enumeration<String> initParameterNames = servletContext.getInitParameterNames();

// 迭代所有参数名和参数值
while (initParameterNames.hasMoreElements()) {
    String paramaterName = initParameterNames.nextElement();
    System.out.println(paramaterName + ": " + servletContext.getInitParameter(paramaterName));
}
```

#### 获取资源文件
* String servletContext.getContextPath()：获取当前Web项目的上下文路径，指的是[部署时Web项目文件夹的名称](tomcat.md#webapps真实目录发布)或者是[使用配置server.xml进行部署时设定的Web项目路径名](tomcat.md#虚拟目录发布)。
* String servletContext.getRealPath(String path)：将当前Web项目部署在文件系统中的路径与方法中的*参数path*进行拼接，返回一个对应于当前文件系统中的绝对路径字符串。如果这个绝对路径是合法的，即使该路径不存在，该方法也正常返回该路径字符串，它被可用于后续基于该路径创建文件或者文件夹等操作。如果这个绝对路径不合法，则该方法返回null。
* String servletContext.getResource(String path)：将当前Web项目部署在文件系统中的路径与方法中的*参数path*进行拼接，返回一个java.net.URL对象，后续可以基于该对象进行读写等操作。如果该java.net.URL对象所代表的路径不存在或者不合法，则该方法返回null。
* InputStream servletContext.getResourceAsStream(String path)：将当前Web项目部署在文件系统中的路径与方法中的*参数path*进行拼接，返回一个指代拼接后的路径的InputStream对象，使用该输入流对象可从指代的文件中读取数据。如果该拼接后的路径无法指向一个存在的文件，那么该方法返回null。

### Cookie技术
Cookie是服务器发送到用户浏览器并存储在本地的小数据文件，浏览器会存储Cookie并在下次向同一服务器再发起请求时携带并发送到服务器上，用于实现状态管理（如登录状态、购物车）、个性化设置和用户行为跟踪。Cookie使基于无状态的HTTP协议能记住用户信息。

#### Cookie的工作原理
1. 首次请求与设置：
   * 用户第一次访问网站时，服务器响应中会包含一个Set-Cookie响应头，里面包含键值对（如session_id=xyz）和过期时间等信息。具体的格式（中括号的部分是可选的）：`Set-Cookie: name=value[; expires=date][; domain=domain][; path=path][; secure][; httpOnly][; sameSime=Lax]`。
   * 浏览器接收到Set-Cookie后，将此数据保存在本地（通常是硬盘上的文本文件或内存）。
2. 后续请求与发送：当用户再次访问同一网站时，浏览器会自动查找并携带该网站的Cookie，将其添加到HTTP请求头（Cookie header）中发送给服务器。例如：`Cookie: user_id=abc123; session=xyz789; theme=dark; lang=zh-CN; _ga=GA1.2.1234567890.1234567890`。
3. 服务器识别与响应：
    * 服务器接收到携带的Cookie，就能识别出是哪个用户（或会话），从而提供个性化服务（记住登录状态或购物车内容）。
    * 服务器也可以根据需要更新Cookie或设置新的Cookie。 

#### Cookie相关的API

##### Cookie的创建和配置 
* Cookie(String name, String value)：构造方法，创建一个Cookie对象。
* String getName()：得到cookie的键。
* String getValue()：得到cookie的值。
* setMaxAge(int expiry)：设置cookie过期时间，单位是秒。
  * 正数：设置秒数；
  * 负数：浏览器关闭就失效；
  * 零：删除cookie。
* setPath(String uri)：设置cookie的访问路径，只有在用户访问这个路径或者它的子路径时，浏览器才会将cookie发送给服务器。

如果用户是通过https访问网页，则还需要对Cookie对象调用setSecure(true)方法，否则浏览器不会发送该Cookie，服务端也就无法获取到该Cookie。

##### 与客户端进行Cookie的交互
* resp.addCookie(Cookie cookie)：利用HttpServletResponse对象，将服务器创建的cookie通过响应发送给浏览器。
* Cookie[] req.getCookies()：利用HttpServletRequest对象，获取浏览器发送过来的所有的cookie信息，该方法返回的是一个Cookie的对象数组。如果没有读到任何的Cookie，返回null。

#### Cookie的编码问题
在Cookie值中不能使用分号（;）、逗号（,）、等号（=）以及空格。Tomcat8中可以直接使用汉字，Tomcat7中不能直接使用汉字。

如果要使用特殊字符，必须在写入字符值之前进行编码。读取以后再进行解码：
* URLEncoder.encode("字符串", "utf-8")：把字符串使用utf8进行编码。
* URLDncoder.dncode("字符串", "utf-8")：把字符串使用utf8进行解码。


### Session会话域
Cookie在客户端存储会话数据，Session在服务端存储会话数据。

Cookie属于客户端的会话技术，数据保存在客户端的文件中，Cookie中键和值都是String类型。

Session属于服务器端的会话技术，数据保存在服务器的内存中，Session中键是String，值是Object类型。服务器会为每个浏览器（每个用户）都会创建独享的Session对象。

#### Session的实现方式
Session技术通常是依赖Cookie技术（Set-Cookie: JSESSIONID=会话ID）实现的，但还有[其他实现方式](https://www.zhihu.com/question/35307626)。

![servlet+20210722165512](https://raw.githubusercontent.com/loli0con/picgo/master/images/servlet%2B20210722165512.png%2B2021-07-22-16-55-16)

#### Cookie和Session的对比
|特点|Cookie|Session|
|---|---|---|
|数据存储在哪一端|客户端|服务器|
|数据大小是否有限制|4kb|没有|
|存储存储什么类型数据|String|Object|
|存储的数据默认有效期|会话结束|会话结束或者是非活动时间30分钟|
|存储的数据是否安全|不安全|安全|

#### Session相关的API
* HttpSession req.getSession()：通过请求对象获得会话对象，不存在就创建，存在就返回。
* boolean isNew()：判断是否为新创建的会话。
* String getId()：会话ID，在服务器上唯一的32位的十六进制数。会话ID指的是`Set-Cookie: JSESSIONID=会话ID`中JSESSIONID的值。
* long getCreationTime()：该会话创建的时间，返回1970-1-1到这个时间的之间相差的毫秒数。
* long getLastAccessedTime()：客户端在携带该会话的情况下，最后一次访问服务器的时间，返回1970-1-1到这个时间的之间相差的毫秒数。
* int getMaxInactiveInterval()：得到该会话最大非活动时间间隔，默认是1800秒（30分钟）。如果客户端与服务器在这个最大非活动时间间隔内未进行过交互（交互的过程中必须携带该会话），则服务器会认为该会话已超时，并使该会话失效。
* setMaxInactiveInterval(int interval)：设置会话最大非活动时间间隔，单位是秒。通过设置该时间间隔，可以改变该会话的超时时间。如果参数*interval*不是一个正数，则该会话被设为永不超时。
* invalidate()：让会话立刻失效，一般用于用户退出注销。

#### Session的默认超时时间
Session在服务器内存中不是永久的，默认距离上一次请求间隔超过30分钟会失效。

可以在web.xml统一修改所有Session超时时间（*Tomcat目录中的conf/web.xml*或者*当前Web项目中的WEB-INF/web.xml*）：
```xml
<session-config>
    <!-- 单位是分钟 -->
    <session-timeout>30</session-timeout>
</session-config>
```

#### 修改JSESSIONID的有效期
在服务端中为某个客户端生成的Session对象之后，服务器通过响应头`Set-Cookie: JSESSIONID=会话ID`将Session对象以Cookie的形式，与该客户端进行绑定（因为客户端下一次会在请求头内回传`Cookie: JSESSIONID=会话ID`）。

但在浏览器关闭的时候，<u>该Cookie数据"JSESSIONID"</u>就会过期。如果再次打开浏览器并访问服务器时，浏览器不会携带<u>该Cookie数据"JSESSIONID"</u>。

如果想要修改<u>该Cookie数据"JSESSIONID"</u>的有效期，让浏览器关闭后再次打开时，也能携带<u>该Cookie数据"JSESSIONID"</u>，则需要通过以下的代码实现：
```Java
// 1. 获得Session对象
HttpSession session = req.getSession();

// 2. 获得sessionId
String sessionId = session.getId();

// 3. 创建Cookie写入JSESSIONID数据
Cookie cookie = new Cookie("JSESSIONID", sessionId);

// 4. 设置JSESSIONID的过期时间
cookie.setMaxAge(60 * 60); // 传参单位是秒，这里示例是60*60*24秒=1小时

// 5. 覆盖默认的cookie数据"JSESSIONID"
resp.addCookie(cookie);
```

通过上述代码，可以修改的Cookie数据"JSESSIONID"的有效期，可以在浏览器（或者接口测试工具）中查看服务器返回的响应头包含如下内容（假定当前时间为2025-12-29 04:14:59 GMT）：
```
Set-Cookie: JSESSIONID=7871A456FC86497CA648BC76F5F6417B; Max-Age=3600; Expires=Mon, 29 Dec 2025 05:14:59 GMT
```

#### Session的钝化与激活
Session钝化是指当服务器正常关闭时，将内存中未过期的Session对象通过序列化成文件（如SESSIONS.ser）存储到磁盘上，以便服务器重启后能反序列化恢复这些Session，保持用户状态不丢失。这个过程需要Session中的对象实现java.io.Serializable接口，并且通常在Tomcat等Servlet容器中配置实现。默认存储位置：$CATALINA_HOME\work\Catalina\localhost\项目名\SESSIONS.ser。

当我们把Web应用程序部署以后，每一个会话的开启，服务端都要为其创建一个Session对象。如果Web应用的访问量比较大，那么很快服务器中就会有大量的Session对象存在，服务器的内存压力会迅速上升。如果能把不活跃的Session对象钝化保存到磁盘文件上，那么服务器就可以腾出内存空间，支持更多的访问了。

##### Session钝化与活化的管理机制
Tomcat提供了两种Session钝化与活化的管理机制：
* StandardManager：StandardManager是Tomcat的默认Session管理机制。
  * 当Tomcat被关闭，或者重启时，会把Session对象钝化保存到磁盘文件上；
  * 当Tomcat启动时，会把保存在磁盘上的文件进行反序列化，恢复到内存中形成一个Session对象；
  * 强制kill掉Tomcat，是不会把Session钝化到磁盘上的。
* PersistentManager：PersistentManager需要进行配置才会生效。配置这种方式，可以将长时间不用的Session对象钝化到磁盘上，减少内存的占用。

##### Session钝化与活化的配置方式

###### 对当前Web应用生效
在当前Web应用的webapp/META-INF目录下创建一个context.xml文件，内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <!--
        className：指定Session管理对象是PersistentManager
        maxActiveSessions：最大活跃Session数，超过将抛出异常，默认为-1（无限）
        maxIdleBackup：最大空闲备份时间（秒），空闲时间超过指定时间时将Session备份到Store，默认为-1（关闭）
        maxIdleSwap：最大空闲清理时间（秒），空闲时间超过指定时间时将Session备份到Store并从内存中清除，默认为-1（关闭）
        minIdleSwap：最小空闲清理时间（秒），空闲时间超过指定时间时才允许清理以减少最大活跃Session数，默认为-1（关闭）
        persistAuthentication：备份Principal认证信息，默认为false
        processExpiresFrequency：备份/清理/失效等后台操作执行周期（单位10秒），默认为6（即60秒）
        saveOnRestart：重启容器时自动备份与恢复Session，默认为true
    -->
    <Manager className="org.apache.catalina.session.PersistentManager" maxIdleSwap="1800">

        <!-- 
            存储到文件夹中
            className：指定钝化Session时生成文件的管理类    
            directory：指定生成的文件保存的文件夹（$CATALINA_HOME\work\Catalina\localhost\web应用名称\指定的文件夹名称）
        -->
        <!-- 存储到文件夹中 -->
        <Store className="org.apache.catalina.session.FileStore" directory="指定的文件夹名称"/>
        <!--
            存储到数据库中
            sessionTable：Session备份表名，默认为tomcat$sessions
            dataSourceName：JNDI数据源
        -->
        <Store className="org.apache.catalina.session.JDBCStore" connectionName="用户名" connectionPassword="密码" connectionURL="jdbc:mysql://127.0.0.1:3306/test" driverName="com.mysql.cj.jdbc.Driver"/>
    </Manager>
</Context>
```

###### 对某个Web应用生效
修改Tomcat的conf/server.xml，在Host标签中增加子标签内容如下：
```xml
<!-- 
    docBase：部署的web应用的路径，可以写web应用的绝对路径，也可以写webapps中的web应用文件夹名称
    path：web应用的访问路径，也就是contextPath 
-->
<Context docBase="activation" path="/activation" reloadable="true">
    <Manager className="org.apache.catalina.session.PersistentManager" maxIdleSwap="1800">
        <Store className="org.apache.catalina.session.FileStore" directory="my_session"/>
    </Manager>
</Context>
```
Context标签中的docBase，配置的哪个web应用，就对哪个Web应用生效。

###### 对所有Web应用生效
修改Tomcat的conf\context.xml，在Context标签里增加子标签内容如下：
```xml
<Manager className="org.apache.catalina.session.PersistentManager" maxIdleSwap="1800">
    <Store className="org.apache.catalina.session.FileStore" directory="my_session"/>
</Manager>
```

#### 禁用Cookie的解决方案
除了使用Cookie机制可以实现Session外，还可以通过隐藏表单、URL末尾附加ID来追踪Session。

##### URL方式
重定向的url重写，把会话的ID在地址栏上带给服务器：`res.encodeURL(path)`。

```Java
// 1.创建session
HttpSession session = req.getSession();

// 2.写入数据
session.setAttribute("username", "admin");

//使用url重写解决
String url = resp.encodeURL(req.getContextPath() + "/Servlet2");
/* 
实现原理：
就是在这个路径req.getContextPath() + "/Servlet2" 后面拼接 “;jsessionid=sessionid”

最终得到：
xxx/Servlet2;jsessionid=4C927EA8EBEBBCE600B6444772BF7951
*/

// 3.跳转到Servlet2去读取展现
resp.sendRedirect(url);
```