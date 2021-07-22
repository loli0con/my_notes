# Filter
javaee体系提供一个Filter接口，主要用于拦截请求与过滤响应。

## 执行流程
![filter+20210722094748](https://raw.githubusercontent.com/loli0con/picgo/master/images/filter%2B20210722094748.png%2B2021-07-22-09-47-50)

## 开发方式
### 注解
```java
// 注解语法：@WebFilter(filterName = "设置过滤器的名名字",urlPatterns = {"拦截的资源路径1", "拦截的资源路径2", ...})
@WebFilter("/DemoServlet")
public class DemoFilter implements Filter {
     /*
     * 初始化方法：当前服务器创建过滤器对象之后调用的方法，只会调用1次，服务器启动时调用
     * 作用：做一些初始化的操作
     * */
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    /*
     * 拦截请求与响应方法：每次浏览器访问拦截路径都会执行
     * */
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        //1.拦截请求
        doSomethingBefore(request,response);
        // System.out.println("==执行了DemoFilter拦截请求的代码==");

        //2.放行(将请求放行给到目标资源去执行)
        filterChain.doFilter(request,response);

        //3.拦截响应
        doSomethingAfter(request,response);
        // System.out.println("==执行了DemoFilter拦截响应的代码==");
    }

    /*
     * 销毁方法：当服务器关闭之前调用的方法，执行1次
     * 作用：做一些销毁的操作
     * */
    @Override
    public void destroy() {

    }
}
```

### web.xml
```xml
<!--filter 标签用于配置一个 Filter 过滤器-->
<filter>
    <!--给 filter 起一个别名-->
    <filter-name>filter名字</filter-name>
    <!--配置 filter 的全类名-->
    <filter-class>filter类全名</filter-class>
</filter>

<!--filter-mapping 配置 Filter 过滤器的拦截路径-->
<filter-mapping>
    <!--表示当前的拦截路径给哪个 filter 使用-->
    <filter-name>filter名字</filter-name>
    <!--配置拦截路径-->
    <url-pattern>拦截的资源路径1</url-pattern>
    <!-- eg：<url-pattern>/DemoServlet</url-pattern> -->
    <url-pattern>拦截的资源路径2</url-pattern>
    ...
</filter-mapping>
```

### 路径映射配置
拦截路径配置方式有2种：
1. 精确匹配，配置的路径与资源访问的路径要一模一样就可以拦截， 任何资源路径都是可以的
   * urlPatterns = "/img/3.jpg",
   * urlPatterns = "/index.jsp"
2. 模糊匹配,只能使用一个“*”号通配符操作，代表0~多个字符
   1. 前缀匹配，匹配开头一致的，要求：以“/”开头，以"/*"结尾，例子：
      * /abc/*,拦截资源访问路径以/abc开头的所有资源
      * /*,拦截所有资源
   2. 后缀匹配,匹配结尾一致的，要求：以“*”开头，以“.扩展名”方式结尾，例子：
       * *.action,拦截资源访问路径以.action为结尾的所有资源
       * *.do,拦截资源访问路径以.do为结尾的所有资源

Filter 过滤器它只关心请求的地址是否匹配，不关心请求的资源是否存在!!!

## 生命周期
Filter 的生命周期包含几个方法：
1. 构造器方法
2. init 初始化方法
   * 启动 web 工程时执行(即Filter创建于服务器启动时)
   * 过滤器对象只创建1次，因为init方法只被调用了1次
   * 过滤器对象是单例对象，全局唯一，节省内存资源
3. doFilter 过滤方法
   * 每次拦截到请求，就会执行 
4. destroy 销毁方法
   * 停止 web 工程的时执行(即Filter销毁于服务器关闭时)


## FilterConfig
FilterConfig是 Filter 过滤器的配置文件类。Tomcat 每次创建 Filter 的时候，也会同时创建一个 FilterConfig 类，这里包含了 Filter 配置文件的配置信息。

### 作用
FilterConfig 类的作用是获取 filter 过滤器的配置内容：
1. 获取 Filter 的名称 filter-name 的内容
2. 获取在 Filter 中配置的 init-param 初始化参数
3. 获取 ServletContext 对象

### 示例
#### web.xml
```xml
<filter>
    <filter-name>DemoFilter</filter-name>
    <filter-class>xxx.xxx.DemoFilter</filter-class>

    <init-param>
        <param-name>username</param-name>
        <param-value>root</param-value>
    <init-param>

    <init-param>
        <param-name>password</param-name>
        <param-value>root</param-value>
    <init-param>
</filter>
```
#### DemoFilter
```java
public class DemoFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 1、获取 Filter 的名称 filter-name 的内容
        System.out.println("filter-name 的值是:" + filterConfig.getFilterName());
        // 2、获取在 web.xml 中配置的 init-param 初始化参数
        System.out.println("初始化参数username的值是:" + filterConfig.getInitParameter("username"));
        System.out.println("初始化参数 password 的值是:" + filterConfig.getInitParameter("password"));
        // 3、获取 ServletContext 对象 
        System.out.println(filterConfig.getServletContext());
    }

    ...
}
```


## 拦截方式
filter默认不会拦截转发的目标资源，可以修改**拦截方式**使filter拦截请求转发的目标资源。

### 拦截方式的类型
|类型|介绍|
|---|---|
|REQUEST|默认拦截方式，过滤器只拦截浏览器直接url访问目标资源的请求与响应|
|FORWARD|过滤器只拦截服务器内部请求转发跳转的目标资源的请求与响应|
|ERROR|（扩展知识）拦截错误友好页面|
|INCLUDE|（扩展知识）拦截包含的页面|


### 修改拦截方式
#### 注解
|@WebFilter注解属性|说明|
|---|---|
|dispatcherTypes = {DispatcherType.REQUEST}|设置采用默认拦截方式，拦截浏览器直接url访问的资源请求与响应|
|dispatcherTypes = {DispatcherType.FORWARD}|设置采用请求转发拦截方式，注意,如果只配置这种方式，默认方式直接url访问资源将无效不能拦截|
|dispatcherTypes = {DispatcherType.REQUEST,DispatcherType.FORWARD}|设置采用直接url与请求转发2种拦截方式|
#### web.xml
```xml
...
<filter-mapping>
    ...
    <!-- xml中配置拦截方式 -->
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>REQUEST</dispatcher>
    ...
</filter-mapping>
```

## filter链
过滤器链：允许一个资源被多个过滤器拦截，过滤器一个接一个串行执行。

![filter+20210722171836](https://raw.githubusercontent.com/loli0con/picgo/master/images/filter%2B20210722171836.png%2B2021-07-22-17-18-39)

### 注解
```java
// 资源
@WebServlet("/DemoServlet")
public class DemoServlet extends HttpServlet {...}


// filter1
@WebFilter("/DemoServlet")
public class Chain1Filter implements Filter {...}

// filter2
@WebFilter("/DemoServlet")
public class Chain2Filter implements Filter {...}
```

执行顺序：通过过滤器类的名字（Chain1Filter < Chain2Filter）升序继续排序。

### web.xml
```xml
<!--配置过滤器链：先执行Chain2Filter，再执行Chain1Filter
    注意：xml配置过滤器，执行顺序就是配置的顺序
-->
<filter>
    <filter-name>Chain2Filter</filter-name>
    <filter-class>...</filter-class>
</filter>
<filter-mapping>
    <filter-name>Chain2Filter</filter-name>
    <url-pattern>/DemoServlet</url-pattern>
</filter-mapping>

<filter>
    <filter-name>Chain1Filter</filter-name>
    <filter-class>...</filter-class>
</filter>
<filter-mapping>
    <filter-name>Chain1Filter</filter-name>
    <url-pattern>/DemoServlet</url-pattern>
</filter-mapping>
```

执行顺序：按照过滤器配置的顺序执行。

如果filter配置了注解（即存在两个顺序），则该注解必须使用属性filterName，才能使xml配置的执行顺序覆盖注解中配置的执行顺序。


## 常见示例

### 全站post提交 编码设置
```java
@WebFilter("/*")
public class EncodingFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {

        //1.获取请求数据的类型
        //req.getMethod() ServletRequest接口中没有getMethod()方法，子接口HttpServletRequest才有
        //将父接口转换为子接口
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        String method = request.getMethod(); //返回get或post

        //2.如果类型为post处理乱码
        if("post".equalsIgnoreCase(method)){
            request.setCharacterEncoding("utf-8");
        }

        //3.放行
        chain.doFilter(req, resp);
    }
}
```