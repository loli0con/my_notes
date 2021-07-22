# JSP

## 前言
本篇笔记有点拉垮。🌿！

## 介绍
JSP(Java Server Pages)：是一种动态网页技术标准。

JSP 部署在服务器上，可以处理客户端发送的请求，并根据请求内容动态的生成 HTML 文档的 Web 网页，然后再响应给客户端。

JSP 是基于 Java 语言的，它的本质就是 Servlet（HttpServlet的子类）。

JSP 实现了布局 Java 代码动态数据的服务器端页面模板技术。

## 原理
![jsp+image-20210324202048734](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2Bimage-20210324202048734.png%2B2021-07-21-14-44-25)

## 语法 - 脚本Java代码

### Java脚本代码片段
![jsp+1566217829332](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B1566217829332.png%2B2021-07-21-14-47-01)
代码片段生成的位置：翻译成java文件在service处理请求方法内部

用途：处理业务逻辑

### Java脚本表达式
![jsp+20210721144841](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721144841.png%2B2021-07-21-14-48-41)
代码片段生成的位置：翻译成java文件在service处理请求方法内部

用途：输出数据

### Jsp脚本声明
![jsp+20210721144948](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721144948.png%2B2021-07-21-14-49-48)
代码片段生成的位置：翻译成java文件在service处理请求方法外部

用途：定义全局成员

### 示例
```Jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--
jsp页面编写java代码的3种方式(脚本代码方式)
1.java脚本代码片段
2.java脚本表达式
3.java脚本声明
--%>

<h1>1.java脚本代码片段</h1>
<%--
格式: <% java代码; %>
作用: 编写的脚本代码生成在servlet代码中service方法内部局部代码
特点: 脚本代码片段在一个jsp页面上可以写多个
--%>
<%--
目标：使用table表格标签布局输出一个List<String>集合数据
--%>
<%
    //1.使用List<String>定义一个集合数据
    List<String> userList = new ArrayList<>();
    userList.add("播仔");
    userList.add("播妞");
%>
<%--//2.使用table布局数据 --%>
<table border="1px" width="50%" align="center">
    <tr>
        <th>序号</th>
        <th>姓名</th>
    </tr>
    <%
        for(int i=0;i<userList.size();i++){
    %>
    <tr>
        <th><% out.print(i+1);  %></th>
        <th><% out.print(userList.get(i)); %></th>
    </tr>
    <%
        }
    %>
</table>


<h1>2.java脚本表达式</h1>
<%--
格式: <%= java的变量或具有返回值的方法 %>
作用: 用于输出数据的,代替out.print(变量);
特点: 脚本代码片段在一个jsp页面上可以写多个
注意: 里面的代码不能写";"
--%>
<table border="1px" width="50%" align="center">
    <tr>
        <th>序号</th>
        <th>姓名</th>
    </tr>
    <%
        for(int i=0;i<userList.size();i++){
    %>
    <tr>
        <th><%= i+1   %></th>
        <th><%= userList.get(i) %></th>
    </tr>
    <%
        }
    %>
</table>

<h1>3.java脚本声明</h1>
<%--
格式: <%! 定义全局的成员(属性字段或方法) %>
作用: 定义全局成员, 生成代码在service方法的外部, 类的内部
特点: 脚本代码片段在一个jsp页面上可以写多个
示例：定义求和方法并返回结果，调用方法并打印输出结果
--%>
<%!
    //定义求和方法
    private int sum(int a,int b){
        return a+b;
    }
%>
<p>
    1+2 = <%= sum(1,2) %>
</p>
</body>
</html>
```

## 语法 - page指令
page 是 jsp 中一个基础指令，用于设置 JSP 上各种页面的属性，一般建议放在页面的最顶部。

![jsp+20210721150609](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721150609.png%2B2021-07-21-15-06-10)

### 示例
```Jsp
<!-- 导包相关 -->
<%@page import="java.util.Date"%>

<!-- 编码相关 -->
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ page language="java" contentType="text/html;charset=UTF-8"%>
<!-- 上述两者等价 -->

<!-- 错误相关 -->
<%@ page isErrorPage="true"%>
<!-- 表示该页面是一个错误页面，这个页面可以使用一个exception对象，可以打印错误消息到控制台，可以给客户端显示友好的提示信息 -->
```

### 错误页面
通过修改web.xml来配置上述提到的错误页面。
```xml
<!--配置错误状态码友好页面:
    作用：当发生错误的时候进行捕捉跳转到友好页面给用户显示
    捕获500，跳转到一个/error/500.jsp页面
    捕获404，跳转到一个/error/404.jsp页面
-->

<error-page>
    <error-code>500</error-code>
    <location>/error/500.jsp</location>
</error-page>
<error-page>
    <error-code>404</error-code>
    <location>/error/404.jsp</location>
</error-page>
```

## 语法 - include指令
include 指令用于在一个 JSP 页面中静态包含另一个 JSP 。

语法：
```Jsp
<%@include file="URL" %>
<!-- 其中URL指定要导入页面的地址 -->
```

静态包含：如果是A包含B，将B页面的内容直接放入到A页面内容中合在一起提供访问，服务器只会翻译出一个A的页面java文件。此时A页面与B页面可以共享变量。

![jsp+20210721151101](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721151101.png%2B2021-07-21-15-11-02)

## 语法 - jsp的动态标签
动态标签的作用相当于java代码作用，sun公司将特定的java代码封成了动态标签，因为编写标签代码更加优美与简洁。

![jsp+20210721151330](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721151330.png%2B2021-07-21-15-13-31)

### include
include 用于一个 JSP 页面动态包含另一个 JSP 页面。

语法：
```jsp
<jsp:include page="URL"/>
<!-- 其中URL是被包含的JSP页面 -->
```

动态包含：是将2个页面分别翻译成2个java文件，再编译运行的结果合并在一起，由于是2个java文件，所以不共享变量。

#### 区别
![jsp+20210721151621](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721151621.png%2B2021-07-21-15-16-22)

### forward
forward用于页面的转发，与`request.getRequestDispatcher("/URL").foward(request,response);`功能一样。

语法：
```jsp
<jsp:forward page"URL"/>
```

### param
param用于页面的转发跳转的时候传递参数数据，经常与`<jsp:forward>`配合使用。

语法：
```jsp
<jsp:param name="参数名" value="参数值"/>
```

#### 示例
one.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%--目标:转发跳转使用param标签传递参数，注意：jsp的动作标签的复合标签形式中间不能有jsp注释,否则发生异常--%>
<jsp:forward page="two.jsp">
    <jsp:param name="username" value="admin"/>
    <jsp:param name="password" value="123456"/>
</jsp:forward>
```

two.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<p>username=<%=request.getParameter("username") %></p>
<p>password=<%=request.getParameter("password") %></p>
```

## 九大内置对象
在jsp页面上可以直接使用的对象（不需要new），就是内置对象。

![jsp+20210721152433](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721152433.png%2B2021-07-21-15-24-34)

## 四个内置域对象
|对象|域|说明|
|---|---|---|
|pageContext|页面域|在一个页面内（一个资源内）有效|
|request|请求域|一次请求内有效，适合转发传递数据|
|session|会话域|一次会话内有效，应用场景存储验证码和用户登录数据|
|servletContext（application别名）|上下文域|整个应用程序有效，存储的数据所有用户共享，应用场景统计某个资源访问量等等|

### pageContext
|页面域操作有关的方法|说明|
|---|---|
|void setAttribute(String key, Object value)|向页面域中添加键和值|
|Object getAttribute(String key)|从页面域中得到值|
|void removeAttribute(String key)|删除页面域中键值对|
|Object findAttribute(String key)|自动从四个作用域中去查某个键，从小到大的范围来查找，如果找到就停止。如果没有找到，返回null|

所有的域对象都拥有set/get/remove属性的方法，不同的域的属性并不相同。pageContext可以根据find方法去依次寻找指定的属性。

通过pageContext，可以获取内置对象。

## el表达式
el表达式用于替换页面上的Java脚本表达式，简化编程。

格式：
```jsp
${域对象里面的key或运算表达式}
```
#### 简化示例
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--
需求：设置域对象存储2个数据10,20，之后从域里面获取进行求和运算
--%>
<%
    //设置页面域存储2个数据
    pageContext.setAttribute("one",10);
    pageContext.setAttribute("two",20);
%>

<p>使用脚本表达式求和计算: <%= Integer.parseInt(pageContext.getAttribute("one").toString())+ Integer.parseInt(pageContext.getAttribute("two").toString()) %> </p>
<p>使用EL表达式求和计算：${one + two}</p>

</body>
</html>
```

### 获取指定域对象的数据
![jsp+20210721154217](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721154217.png%2B2021-07-21-15-42-18)

### 获取不同类型的数据
* javaBean数据获取: ${域的key.javaBean对象的属性名}
  * 属性名指的是get**属性名**()方法/set**属性名**()方法中的**属性名**，即访问一个属性不需要get/set前缀。
  * key中不能有特殊字符，使用\["属性名"]的方式去访问，例如`header["user-agent"]`。
* Map集合数据获取: ${域的key.map的key}
* List集合数据获取: ${域的key\[索引]}
* 数组数据获取: ${域的key\[索引]}

### 使用运算符表达式
#### 算术运算符
![jsp+20210721161239](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721161239.png%2B2021-07-21-16-12-40)

#### 比较运算符
![jsp+20210721161308](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721161308.png%2B2021-07-21-16-13-11)

#### 逻辑运算符
![jsp+20210721161428](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721161428.png%2B2021-07-21-16-14-29)

#### 三元运算符
语法：
```jsp
${布尔表达:真值表达式:假值表达式}
```

#### 判空运算符
```jsp
${empty 表达式}
<!-- 表达式为空输出true，否则输出false -->

<!-- 衍生用法 -->
${not empty 表达式}
```

### el中的内置对象
![jsp+20210721162324](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721162324.png%2B2021-07-21-16-23-25)

![jsp+20210721162419](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721162419.png%2B2021-07-21-16-24-20)

#### 常用示例
```jsp
<p>项目路径：${pageContext.request.contextPath}</p>
<%--jsp的pageContext.getRequest().getContextPath()，省略了get前缀--%>


<%
    response.addCookie(new Cookie("username","itheima"));
%>
<p>cookie的name对应的value:${cookie.username.value}</p>

<%-- 比如：url?name=admin&hobby=game&hobby=爱编程 --%>
<p>读取参数名name的数据:${param.name}</p>  <%--admin--%>
<p>读取参数名hobby的第一个爱好数据:${paramValues.hobby[0]}</p><%--game--%>
<p>读取参数名hobby的第二个爱好数据:${paramValues.hobby[1]}</p><%--爱编程--%>
```

## jstl标准标签库
Jsp Standard Tag Library，Jsp标准标签库，apache开源组织开发的。  
jstl标签用于替换Java脚本代码片段，简化编程。

### 组成
![jsp+20210721165643](https://raw.githubusercontent.com/loli0con/picgo/master/images/jsp%2B20210721165643.png%2B2021-07-21-16-56-44)

### 使用方式
#### 导包 
* javax.servlet.jsp.jstl.jar
* jstl-impl.jar

#### jsp页面导入
在jsp页面中使用if/foreach标签：
```jsp
<%--
使用taglib指令导入jstl核心标签库
    prefix="c" 设置使用jstl核心标签里面标签的前缀
    uri="http://java.sun.com/jsp/jstl/core" 指向核心标签库的地址
--%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>



<%--
if标签

格式：<c:if test="${条件表达式}">
        //符合条件运行的代码
      </c:if>
作用：用于条件判断，与java的if功能一样，没有else-if
--%>
<c:if test="${el条件表达式}"> ...为true时的内容... </c:if>



<%--
格式：
    <c:forEach begin="" end="" var="" step="">
      //循环体代码
    </c:forEach>
属性介绍
    begin：循环开始值
    end：循环结束值
    var：循环变量名称,会被存入到pagecontext域中，便于循环体获取使用
    step：步长，就是循环的递增值,默认就是1
示例：foreach标签遍历5次循环写法
--%>
<c:forEach begin="1" end="5" step="1" var="i">
${i}
</c:forEach>

<%--
格式：
    <c:forEach items="" var="" varStatus="">
        //循环体代码
    </c:forEach>
属性介绍
    items,遍历的集合，使用el获取
    var，循环元素对象的名字,会被存入到pagecontext域中，便于循环体获取使用
    varStatus， 循环过程信息对象名字，并且放入了域对象中，里面有循环的序号，索引等
           varStatus="status",循环序号：  ${status.count}  从1开始
           varStatus="status",循环索引：  ${status.index}  从0开始
示例：定义List集合，遍历集合
--%>
<c:forEach items="${list}" var="item" varStatus="status">
    ${status.index} of ${status.count} is: ${item}, ${item.attribute}
</c:forEach>

<%--
格式：
    <c:forEach items="" var="" varStatus="">
        //循环体代码
    </c:forEach>
属性介绍
    items,遍历的map集合(本质遍历Set<Map.Entry<String, User>>集合)，使用el获取
          eg:Set<Map.Entry<String, User>> entries = map.entrySet();
    var，就是Map.Entry<String, User>对象的名字，获取数据,var="entry"
         获取key:${entry.key}
         获取value:${entry.value}
示例：定义Map集合，遍历集合
--%>
<c:forEach items="${map}" var="entry">
${entry.key}, ${entry.value}, ${entry.value.attribute}
</c:forEach>
```

