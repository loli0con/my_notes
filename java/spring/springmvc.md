# SpringMVC
Spring MVC是Spring Framework的一部分，是基于Java实现MVC的轻量级Web框架；  
它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口；  
同时它还支持RESTful 编程风格的请求。

![springmvc+20210801120809](https://i.loli.net/2021/08/01/ri85DZILefMl9oO.png)

## 优点
| 序号 | 优点                   | 详情                                                                                                                                                                                                                                                                                                      |
| ---- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | 清晰的角色划分         | 前端控制器（DispatcherServlet）<br/>处理器映射器（HandlerMapping）<br/>处理器适配器（HandlerAdapter）<br/>视图解析器（ViewResolver）<br/>处理器（Controller）<br/>验证器（Validator）<br/>命令对象（Command 请求参数绑定到的对象就叫命令对象）<br/>表单对象（Form Object 提供给表单展示和表单提交的对象） |
| 2    | 可扩展性好             | 可以很容易扩展，虽然几乎不需要                                                                                                                                                                                                                                                                            |
| 3    | 与Spring 框架无缝集成  | 这是其它web框架不具备的                                                                                                                                                                                                                                                                                   |
| 4    | 可适配性好             | 通过 HandlerAdapter 可以支持任意的类作为处理器                                                                                                                                                                                                                                                            |
| 5    | 可定制性好             | 处理器映射器HandlerMapping和<br />视图解析器ViewResolver 能够非常简单的定制                                                                                                                                                                                                                               |
| 6    | 单元测试方便           | 能够非常简单的进行 Web 层单元测试                                                                                                                                                                                                                                                                         |
| 7    | 本地化、主题的解析支持 | 使我们更容易进行国际化和主题的切换                                                                                                                                                                                                                                                                        |
| 8    | JSP标签库              | 强大的 JSP 标签库，使 JSP 编写更容易                                                                                                                                                                                                                                                                      |

## 架构
Spring的web框架围绕DispatcherServlet（调度Servlet）设计，DispatcherServlet的作用是将请求分发到不同的处理器。

![springmvc+20210801094532](https://i.loli.net/2021/08/01/jQpvqVntOZWFUgo.png)

### 组件
#### DispatcherServlet 前端控制器
作用：接收请求，响应结果，相当于转发器，中央处理器。  
DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。

#### HandlerMapping 处理器映射器
作用：根据请求的url查找Handler  
HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等

#### HandlerAdapter 处理器适配器
作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler  
通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。（因为不同的处理器有不同的实现方式，有注解的方式实现的，有配置文件的实现的。）

#### Handler 处理器
Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。
由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

#### ViewResolver 视图解析器
作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后返回视图对象。




## FirstDemo
这个Demo很标准地展示了SpringMVC的原理，但实际开发中不会这么写，有更简单的写法（注解版）。
### web.xml
在web.xml中配置DispatcherServlet前端控制器，接管所有的请求。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
        version="4.0">

    <!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
        <!-- 关联一个springmvc的配置文件（名字自取，统一即可）: springMVC.xml-->
        <init-param>
            <!-- contextConfigLocation为固定值 -->
            <param-name>contextConfigLocation</param-name>
            <!-- 使用classpath:表示从类路径查找配置文件 -->
            <param-value>classpath:springMVC.xml</param-value>
        </init-param>
        <!--启动级别1：服务器启动时就创建中央控制器-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- / 所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求 -->
        <!-- /* 匹配所有的请求（包括.jsp）-->
        <url-pattern>/</url-pattern>

        <!--配置DispatcherServlet处理.do结尾的请求-->
        <!-- <url-pattern>*.do</url-pattern> -->
    </servlet-mapping>

</web-app>
```

### springMVC.xml
将HandlerMapping 处理器映射器、HandlerAdapter 处理器适配器、Handler 处理器、ViewResolver 视图解析器注册到SpringIOC容器中。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 处理器映射器 -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!-- 处理器适配器 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
    
    <!--在SpringIOC容器，注册bean；这是自己编写的 Handler 处理器；id参数定义了要处理的请求的路径-->
    <bean id="/hello" class="com.demo.controller.HelloController"/>

</beans>
```

### HelloController
```java
public class HelloController implements Controller {
   public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
       //ModelAndView 模型和视图
       ModelAndView mv = new ModelAndView();

       //封装对象，放在ModelAndView中。Model
       mv.addObject("msg","HelloSpringMVC!");
       //封装要跳转的视图，放在ModelAndView中
       mv.setViewName("hello"); //被视图解析器拼接称 /WEB-INF/jsp/hello.jsp
       return mv;
  }
}
```

### hello.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
   <title>Demo</title>
</head>
<body>
${msg}
</body>
</html>
```




## SecondDemo
对FirstDemo进行简化，描述了实际开发中代码：需要手动配置视图解析器，而处理器映射器和处理器适配器只需要开启注解驱动即可，而省去了大段的xml配置。

### springMVC.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:mvc="http://www.springframework.org/schema/mvc"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

   <!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
   <context:component-scan base-package="com.demo.controller"/>
   <!-- 让Spring MVC不处理静态资源 -->
   <mvc:default-servlet-handler />
   <!--
   支持mvc注解驱动
       在spring中一般采用@RequestMapping注解来完成映射关系
       要想使@RequestMapping注解生效
       必须向上下文中注册DefaultAnnotationHandlerMapping
       和一个AnnotationMethodHandlerAdapter实例
       这两个实例分别在类级别和方法级别处理。
       而annotation-driven配置帮助我们自动完成上述两个实例的注入。
    -->
   <mvc:annotation-driven />

   <!-- 视图解析器 -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
         id="internalResourceViewResolver">
       <!-- 前缀 -->
       <property name="prefix" value="/WEB-INF/jsp/" />
       <!-- 后缀 -->
       <property name="suffix" value=".jsp" />
   </bean>

</beans>
``` 

### HelloController
```java
@Controller
@RequestMapping("/HelloController")
public class HelloController {
   //真实访问地址 : 项目名/HelloController/hello
   @RequestMapping("/hello")
   public String sayHello(Model model){
       //向模型中添加属性msg与值，可以在JSP页面中取出并渲染
       model.addAttribute("msg","hello,SpringMVC");
       return "hello"; //被视图解析器拼接称 /WEB-INF/jsp/hello.jsp
  }
}
```


## Controller
* 控制器提供访问应用程序的行为，通常通过接口定义或注解定义两种方法实现。
* 控制器负责解析用户的请求并将其转换为一个模型。
* 在Spring MVC中一个控制器类可以包含多个方法
* 在Spring MVC中，对于Controller的配置方式有很多种.

### 实现Control
#### 实现Controller接口 - 不推荐
Controller是一个接口，在org.springframework.web.servlet.mvc包下，接口中只有一个方法：

```java
//实现该接口的类获得控制器功能
public interface Controller {
   //处理请求且返回一个模型与视图对象
   ModelAndView handleRequest(HttpServletRequest var1, HttpServletResponse var2) throws Exception;
}
```

实现接口Controller定义控制器是较老的办法；一个控制器中只有一个方法，如果要多个方法则需要定义多个Controller；定义的方式比较麻烦。

#### 使用注解@Controller - 推荐
Spring使用扫描机制来找到应用程序中所有基于注解的控制器类，为了保证Spring能找到你的控制器，需要在配置文件中声明组件扫描。

```xml
<!-- 自动扫描指定的包，下面所有注解类交给IOC容器管理 -->
<context:component-scan base-package="com.demo.controller"/>
```

@Controller注解类型用于声明Spring类的实例是一个控制器：
```java
//@Controller注解的类会自动添加到Spring上下文中
@Controller
public class IndexController{
   //映射访问路径
   @RequestMapping("/index")
   public String index(Model model){
       //Spring MVC会自动实例化一个Model对象用于向视图中传值
       model.addAttribute("msg", "Hello World!");
       //返回视图位置
       return "index";
  }
```

### 路径处理
#### @RequestMapping - 路径映射配置
@RequestMapping注解将请求和处理请求的控制器关联起来，建立映射关系。可用于控制器的类或方法上。用于类上时，表示类中的所有响应请求的方法都是以该地址作为父路径，举个例子：
1. 配置在方法上
   * 方法映射路径：@RequestMapping("/save.do")
   * 浏览器访问路径：http://localhost:8080/项目路径/save.do
2. 配置在类上和方法上 【推荐，可以区别不同的controller】
   * 类上映射路径：@RequestMapping("/order")
   * 方法映射路径：@RequestMapping("/save.do")
   * 浏览器访问路径： http://localhost:8080/项目路径/order/save.do

在路径中(即value属性)可以使用如下占位符：
* ？：表示任意的单个字符
* *：表示任意的0个或多个字符
* \\**：表示任意的一层或 多层目录（只能使用/\*\*/xxx的方式）

##### 属性详解
@RequestMapping的属性：
* path/value：指定访问地址（value属性是一个字符串类型的数组），不满足value->404
  * 格式1：@RequestMapping("/save.do") 完整写法
  * 格式2：@RequestMapping("/save") 去掉扩展名写法，依然可以映射save.do的请求路径【推荐方式】
  * 格式3：@RequestMapping("save")  省略 / 写法，依然可以映射save.do的请求路径
  * 格式4：@RequestMapping("save.do")
* method：限制请求的方法（method属性是一个RequestMethod类型的数组），满足value不满足method->405
* param：限制提交的参数（params属性是一个字符串类型的数组），满足value&method不满足param->400
  * {"id","name"} **必须有**这两个参数的名字，否则会出现400错误
  * {"!id","!name"} **必须没有**这两个参数的名字，否则会出现400错误
  * {"id=1","name=newboy"} 不但要有这些参数，而且**值还有限制**
  * {"id!=1"} id**不等于**1的值都可以


##### 派生注解
* @GetMapping等价于@RequestMapping(method=RequestMethod.GET) 
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

#### @PathVariable - 路径参数提取
在@RequestMapping注解的value属性中使用占位符`{xxx}`表示传输的数据，再通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参。

```java
@Controller
public class DemoController {
    //映射访问路径
    @RequestMapping("/commit/{p1}/{p2}")
    public String commit(@PathVariable int p1, @PathVariable("p2") int p3, Model model){
        //默认使用根据形参的名字进行匹配并赋值，也通过可以指定value属性来改变默认行为
        int result = p1+p3;
        //Spring MVC会自动实例化一个Model对象用于向视图中传值
        model.addAttribute("msg", "结果："+result);
        //返回视图位置
        return "test";
    }
}
```

### 获取请求参数
本小节描述如何获取请求参数：
* url
  * url的部分（上文）
  * QueryString（本文）
* 请求头
* 请求体

#### 通过ServletAPI获取
在Controller里面，可以直接使用servlet对象：
* 请求对象HttpServletRequest
* 响应对象HttpServletResponse
* 会话对象HttpSession

它们作为方法的参数声明，并由SpringMVC负责注入。

如果要操作会话域session\操作cookie\上下文域\设置响应头\设置响应行数据必须使用原生servletAPI。

使用servlet的原生API推荐控制器方法返回值为void，就不会走视图。

```java
@RequestMapping("/test")
public String test(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
    // String username = request.getParameter("username");
    // String password = request.getParameter("password");
    doSomething(request, response, session);
}
```

#### 控制器方法的形参获取请求参数
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给/绑定到相应的形参。

##### 用到的注解
它们共有的三个属性：value、required、defaultValue
###### @RequestParam
作用：@RequestParam是将请求参数（包括QueryString和请求体中的 x-www-form-urlencoded类型的数据）和控制器方法的形参创建映射关系

| @RequestParam属性 | 应用场景：用于提交的参数名与方法的形参不同的情况 |
| ----------------- | ------------------------------------------------ |
| name/value        | 指定提交的参数名                                 |
| required          | 是否是必须的                                     |
| defaultValue      | 如果没有值，使用这个默认值                       |


###### @RequestHeader
* 作用：用于获取请求头的值（String类型）
* 位置：放在方法的参数前面
* 属性：
  * value：指定请求头的名字

```java
@RequestMapping("/header")
public String header(@RequestHeader("user-agent") String header) {
    System.out.println("请求头：" + header );
    return "success";
}
```

###### @RequestBody
* 作用：用于获取请求体的数据（String类型）
* 位置：放在方法的参数前面
* 注意：只能出现1次，因为每次请求只有一个请求体，只能用于POST请求，因为GET请求没有请求体
* 属性：
  * required：请求体中的数据是否必须

###### @CookieValue
* 作用：用于获取Cookie的值（String类型）
* 位置：放在方法的参数前面
* 属性：
  * value：指定Cookie的名字

```java
@RequestMapping("/cookie")
public String cookie(@CookieValue("JSESSIONID") String cookieValue) {
    System.out.println("获取Cookie的值：" + cookieValue);
    return "success";
}
```

##### 赋值/绑定的规则
SpringMVC参数绑定的规则：
1. Servlet对象：请求，响应，会话
2. 简单类型：直接绑定 (八种基本类型+字符串类型)
3. 简单类型的数组和集合：
   1. 数组：直接绑定
   2. 集合：要设置@RequestParam注解
4. POJO对象类型（可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值）
   1. 普通属性：直接绑定
   2. 简单类型的属性集合：直接绑定
   3. POJO属性集合：对象名\[0].属性名
   4. POJO属性的Map：对象名\['键'].属性名

##### 简单类型
使用包装类可以减少错误的发生：
* 包装类型没有值就为null
* 简单类型没有值会出现500错误（尝试将一个null转成int，所以报错）

1. 提交数据 : http://localhost:8080/hello?name=kuangshen&age=18
2. 处理方法 :
```java
@RequestMapping("/hello")
public String hello(String name, Integer age){
   System.out.println(name+age);
   return "hello";
}
```
3. 后台输出 : kuangshen18

###### 参数名不一致的情况
1. 提交数据 : http://localhost:8080/hello?username=kuangshen
2. 处理方法 : 使用@RequestParam注解
```java
//@RequestParam("username") : username提交的域的名称
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name){
   System.out.println(name);
   return "hello";
}
```
3. 后台输出 : kuangshen

##### 数组/集合
1. 提交数据 : http://localhost:8080/hello?arr=1&arr=2&list=aa&list=bb
2. 处理方法 :
```java
@RequestMapping("/hello")
public String hello(@RequestParam ArrayList<String> list,Integer[] nums) { //需要指定@RequestParam注解在集合形参前
    //输出集合中元素
    System.out.print(list);
    System.out.print(Arrays.toString(nums));
    return "hello";
}
```
3. 后台输出 : \[aa, bb]\[1, 2]

##### 日期类型
1. 提交数据 : http://localhost:8080/search?birth=2011-11-11
2. 处理方法 :
```java
// java.sql.Date可兼容
// java.util.Date会报错
@RequestMapping("/search")
public String search(Date birth) { 
    System.out.print(birth);
    return "hello";
}
```
3. 后台输出 : 2011-11-11

##### POJO类型
请求参数名称要和对象的属性名一致（否则就是null），参数使用对象即可。
1. 实体类
```java
public class User {
   private String name;
   private Integer age;
   //构造
   //get/set
   //tostring()
}
```
2. 提交数据 : http://localhost:8080/user?name=kuangshen&age=15
3. 处理方法
```java
@RequestMapping("/user")
public String user(User user){
   System.out.println(user);
   return "hello";
}
```
4. 后台输出 : User(name=奇魔猪, age=18)

##### 复合POJO类型（类中类）
1. 实体类
```java
public class User {
    private String name;
    private Integer age;
    //实体类中包含了其它的实体类
    private Address address;
    //简单类型的集合
    private List<String> hobby;  //爱好
    //POJO类型的List集合
    private List<Address> addressList;
    //POJO类型的Map集合
    private Map<String,Address> map;
}
class Address {
    private String province;
    private String city;
}
```
2. 提交数据（注意需要*url编码*） : http://localhost:8080/user?name=奇魔猪&age=18&address.province=广东&address.city=广州&hobby=唱歌&hobby=跳舞&addressList\[0].province=浙江&addressList\[0].city=杭州&addressList\[1].province=湖北&addressList\[1].city=武汉&map\["pig"].province=广东&map\["pig"].city=佛山&map\["dog"].province=广东&map\["dog"].city=深圳
3. 处理方法
```java
@RequestMapping("/user")
public String user(User user){
   System.out.println(user);
   return "hello";
}
```
4. 后台输出 : User(id=null, name=奇魔猪, age=18, address=Address(province=广东, city=广州), hobby=[唱歌, 跳舞], addressList=[Address(province=浙江, city=杭州), Address(province=湖北, city=武汉)], map={dog=Address(province=广东, city=深圳), pig=Address(province=广东, city=佛山)})

对象的属性中如果是POJO类型的集合：
* List：list\[0].属性名
* Map：map\['键'].属性名

##### 自定义类型转换器
1. 创建一个类实现Converter接口<源类型(通常为String), 目标类型> 
2. 重写convert方法，进行类型转换（参数是要转换的源对象，返回转换好的目标对象）
3. 配置springMVC.xml文件

###### Demo
```java
// 转换器
public class UserConverter implements Converter<String, User> {
    /**
     * 重写转换方法
     * @param source 源的值
     * @return 目标的对象
     */
    @Override
    public User convert(String source) {
        // 查询字符串为：xxx?user=1000,NewBoy,广东省,广州市

        //封装成user对象
        User user = new User();
        Address address = new Address();
        //将源切割成多个元素
        String[] split = source.split(",");
        //手动封装成对象
        user.setId(Integer.valueOf(split[0]));
        user.setName(split[1]);
        address.setProvince(split[2]);
        address.setCity(split[3]);
        user.setAddress(address);
        return user;
    }
}
```

```xml
<!-- 1. 配置转换服务工厂类ConversionServiceFactoryBean -->
<bean class="org.springframework.context.support.ConversionServiceFactoryBean"  id="factoryBean">
    <!-- 2. 指定converters属性，这是一个set集合，可以引用多个转换器对象。 -->
    <property name="converters">
        <set>
            <bean class="com.itheima.converter.UserConverter"/>
        </set>
    </property>
</bean>
<!-- 3. 注解`mvc:annotation-driven`指定`conversion-service`属性，指定上面的转换器工厂类 -->
<mvc:annotation-driven conversion-service="factoryBean"/>
```

### 数据显示到前端
#### 操作域对象
请求域：
1. 通过Map：用put()方法向请求域中添加键和值
2. 通过Model：用addAttribute()方法向请求域中添加键和值
3. 通过ModelMap：用addAttribute()方法向请求域中添加键和值
4. 通过ModelMap：用addAttribute()方法向请求域中添加键和值
5. 通过ModelAndView：用addAttribute()方法向请求域中添加键和值
6. 通过ServletAPI：直接操作request对象

session域：
1. 通过HttpSession

application域：
1. 通过ServletContext
·

#### demo
```java
@RequestMapping("/sport")
public String sport(Map<String,Object> map) {
    //如果向Map中添加键和值，就相当于向请求域中添加了键和值
    map.put("id", 100);
    map.put("name", "李四");
    System.out.println(map);
    return "success";
}

@RequestMapping("/swim")
public String swim(Model model) {
    //支持链式写法，可以添加多个键和值。也是添加到请求域中
    model.addAttribute("id", 200).addAttribute("name", "孙悟空");
    return "success";
}

@RequestMapping("/draw")
public String draw(ModelMap model) {
    //支持链式写法，可以添加多个键和值。也是添加到请求域中
    model.addAttribute("id", 300).addAttribute("name", "猪八戒");
    return "success";

@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "hello,session");
    return "success";

@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```


### 转发和重定向
```java
@Controller
public class DemoController {
    @RequestMapping("/t1")
    public String test1(){
        //转发
        return "test";
        //return "forward:/test";
    }

    @RequestMapping("/t2")
    public String test2(){
        //重定向
        return "redirect:/index.jsp";
        //return "redirect:hello.do"; //hello.do为另一个请求/
    }
}
```


### Json交互
下面的注解需要jackson包支持，同时在SpringMVC的核心配置文件中开启mvc的注解驱动。
#### @RequestBody
在处理器方法形参上使用，把请求体的json格式数据，转换成java对象。

注解@RequestBody接收的参数是来自requestBody中，即请求体。一般用于处理非 Content-Type: application/x-www-form-urlencoded编码格式的数据，比如：application/json、application/xml等类型的数据。

[和RequestParam的区别](https://blog.csdn.net/weixin_38004638/article/details/99655322)

#### @ResponseBody
在处理器方法上使用@ResponseBody注解进行标识，将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串。

#### Demo
```java
@Controller
public class UserController {

    //produces:指定响应体返回类型和编码(避免乱码)
    @RequestMapping(value="/json2",produces = "application/json;charset=utf-8")
    @ResponseBody
    public User jsonData(@RequestBody User user){
        System.out.println("name:"+user.getName());
        System.out.println("age:"+user.getAge());
        System.out.println("sex:"+user.getSex());
        return user;
    }

    //produces:指定响应体返回类型和编码(避免乱码)
    @RequestMapping(value = "/json1",produces = "application/json;charset=utf-8")
    @ResponseBody
    public String json1() throws JsonProcessingException {
        //创建一个jackson的对象映射器，用来解析数据
        ObjectMapper mapper = new ObjectMapper();
        //创建一个对象
        User user = new User("秦疆1号", 3, "男");
        //将我们的对象解析成为json格式
        String str = mapper.writeValueAsString(user);
        //由于@ResponseBody注解，这里会将str转成json格式返回；十分方便
        return str;
    }

    //Jackson 默认是会把时间转成timestamps形式
    @RequestMapping("/json2")
    @ResponseBody
    public String json2() throws JsonProcessingException {
        ObjectMapper mapper = new ObjectMapper();
        //不使用时间戳的方式
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        //自定义日期格式对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //指定日期格式
        mapper.setDateFormat(sdf);
        Date date = new Date();
        String str = mapper.writeValueAsString(date);
        return str;
    }
}
```
#### @RestController
在类上直接使用 @RestController，那么类中所有的方法都只会返回 json 字符串。

#### JSON乱码统一解决
在springmvc的配置文件上添加一段消息StringHttpMessageConverter转换配置
```xml
<mvc:annotation-driven>
   <mvc:message-converters register-defaults="true">
       <bean class="org.springframework.http.converter.StringHttpMessageConverter">
           <constructor-arg value="UTF-8"/>
       </bean>
       <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
           <property name="objectMapper">
               <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                   <property name="failOnEmptyBeans" value="false"/>
               </bean>
           </property>
       </bean>
   </mvc:message-converters>
</mvc:annotation-driven>
```


### 文件
#### 上传
##### 配置bean
```xml
<!--文件上传配置-->
<!-- id必须为：multipartResolver -->
<bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
   <!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
   <property name="defaultEncoding" value="utf-8"/>
   <!-- 上传文件大小上限，单位为字节（10485760=10M） -->
   <property name="maxUploadSize" value="10485760"/>
   <!-- <property name="maxUploadSize"  value="#{1024*1024*10}"/> -->
   <property name="maxInMemorySize" value="40960"/>
</bean>
```
##### API
MultipartFile(接口)/CommonsMultipartFile(实现类)的常用方法：
* String getOriginalFilename()：获取上传文件的原名
* InputStream getInputStream()：获取文件流
* void transferTo(File dest)：将上传文件保存到一个目录文件中

##### Demo
```java
@RequestMapping("/uploadFile")
public String  uploadFile(@RequestParam("file") MultipartFile file, HttpServletRequest request) throws IOException {

    //1.生成一个文件唯一文件名（防止不同用户上传相同文件名覆盖）
    //文件名格式：xxxx.jpg
    //1.1 生成一个唯一值（使用jdk提供的UUID，通用唯一码，利用系统当前时间+系统中网卡的mac地址进行组合生成的唯一值，生成一个32字符长度的字符串）
    String uuid = UUID.randomUUID().toString();
    //1.2 获取上传文件的扩展名
    String originalFilename = file.getOriginalFilename();//获取上传的文件名,例如：6.jpg
    //1.3 拼接文件完整名
    String fileExtName = originalFilename.substring(originalFilename.lastIndexOf("."));  //获取到：".jpg"
    String fileName = uuid + fileExtName;  //xxxxxxxxxxx.jpg

    System.out.println("上传的唯一文件名："+fileName);

    //2.获取服务器上传文件的位置
    //2.1 定义上传文件的相对路径
    String realPath = "/upload/"+new SimpleDateFormat("yyyy-MM-dd").format(new Date());
    //2.2 使用ServletContext根据相对路径获取绝对路径
    String path = request.getServletContext().getRealPath(realPath);
    //2.3 判断upload和日期目录是否存在，不存在就创建
    File f = new File(path);
    //判断父目录是否存在
    if(!f.getParentFile().exists()){
        //创建父目录
        f.getParentFile().mkdir();
    }
    //判断日期子目录是否存在
    if(!f.exists()){
        f.mkdir();
    }

    //3.实现文件写入到上传文件位置
    file.transferTo(new File(path,fileName));

    return "success";
}
```

#### 文件下载
##### Demo
```java
@RequestMapping(value="/download")
public void downloads(HttpServletResponse response ,HttpServletRequest request) throws Exception{
    //要下载的图片地址
    String path = request.getServletContext().getRealPath("/upload");
    String fileName = "基础语法.jpg";

    //1、设置response 响应头
    response.reset(); //设置页面不缓存,清空buffer
    response.setCharacterEncoding("UTF-8"); //字符编码
    response.setContentType("multipart/form-data"); //二进制传输数据
    //设置响应头
    response.setHeader("Content-Disposition",
            "attachment;fileName="+URLEncoder.encode(fileName, "UTF-8"));

    File file = new File(path,fileName);
    //2、 读取文件--输入流
    InputStream input = new FileInputStream(file);
    //3、 写出文件--输出流
    OutputStream out = response.getOutputStream();

    //  4a/4b 二选一即可

    //4a、执行输出操作（工具包）
    IOUtils.copy(input,out);
    
    //4b、执行写出操作（Java原生）
    byte[] buff =new byte[1024];
    int index=0;
    while((index= input.read(buff))!= -1){
        out.write(buff, 0, index);
        out.flush();
    }
    out.close();
    input.close();
}
```

## RESTful
### HiddenHttpMethodFilter
由于浏览器只支持发送get和post方式的请求，SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**。

**HiddenHttpMethodFilter** 处理put和delete请求的条件：
1. 当前请求的请求方式必须为post
2. 当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数\_method的值才是最终的请求方式。

### 配置
在web.xml中注册**HiddenHttpMethodFilter**：
```xml
<!-- 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```


## 参数乱码问题
解决**获取请求参数**的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter（或者第三方的），记得须在web.xml中进行注册。

SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效。
### Spring提供的过滤器
Spring提供的过滤器，在web.xml中配置
```xml
<!-- 配置spring写的汉字乱码的过滤器 -->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!-- 指定编码 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 自定义过滤器
网上大神写的
```java
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.Map;

/**
* 解决get和post请求 全部乱码的过滤器
*/
public class GenericEncodingFilter implements Filter {

   @Override
   public void destroy() {
  }

   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
       //处理response的字符编码
       HttpServletResponse myResponse=(HttpServletResponse) response;
       myResponse.setContentType("text/html;charset=UTF-8");

       // 转型为与协议相关对象
       HttpServletRequest httpServletRequest = (HttpServletRequest) request;
       // 对request包装增强
       HttpServletRequest myrequest = new MyRequest(httpServletRequest);
       chain.doFilter(myrequest, response);
  }

   @Override
   public void init(FilterConfig filterConfig) throws ServletException {
  }

}

//自定义request对象，HttpServletRequest的包装类
class MyRequest extends HttpServletRequestWrapper {

    private HttpServletRequest request;
    //是否编码的标记
    private boolean hasEncode;
    //定义一个可以传入HttpServletRequest对象的构造函数，以便对其进行装饰
    public MyRequest(HttpServletRequest request) {
        super(request);// super必须写
        this.request = request;
    }

    // 对需要增强方法 进行覆盖
    @Override
    public Map getParameterMap() {
        // 先获得请求方式
        String method = request.getMethod();
        if (method.equalsIgnoreCase("post")) {
            // post请求
            try {
                // 处理post乱码
                request.setCharacterEncoding("utf-8");
                return request.getParameterMap();
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        } else if (method.equalsIgnoreCase("get")) {
            // get请求
            Map<String, String[]> parameterMap = request.getParameterMap();
            if (!hasEncode) { // 确保get手动编码逻辑只运行一次
                for (String parameterName : parameterMap.keySet()) {
                    String[] values = parameterMap.get(parameterName);
                    if (values != null) {
                        for (int i = 0; i < values.length; i++) {
                            try {
                                // 处理get乱码
                                values[i] = new String(values[i]
                                        .getBytes("ISO-8859-1"), "utf-8");
                            } catch (UnsupportedEncodingException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
                hasEncode = true;
            }
            return parameterMap;
        }
        return super.getParameterMap();
    }

    //取一个值
    @Override
    public String getParameter(String name) {
        Map<String, String[]> parameterMap = getParameterMap();
        String[] values = parameterMap.get(name);
        if (values == null) {
            return null;
        }
        return values[0]; // 取回参数的第一个值
    }

    //取所有值
    @Override
    public String[] getParameterValues(String name) {
        Map<String, String[]> parameterMap = getParameterMap();
        String[] values = parameterMap.get(name);
        return values;
    }
}
```


## 拦截器
SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理。拦截器是AOP思想的具体应用。

### 和过滤器的区别
* 过滤器
  * servlet规范中的一部分，任何java web工程都可以使用
  * 在url-pattern中配置了/*之后，可以对所有要访问的资源进行拦截
* 拦截器 
  * 拦截器是SpringMVC框架自己的，只有使用了SpringMVC框架的工程才能使用
  * 拦截器只会拦截访问的控制器方法，如果访问的是jsp/html/css/image/js是不会进行拦截的

### 自定义拦截器
#### 拦截器代码
```java
public class MyInterceptor implements HandlerInterceptor {
    //在控制器的方法之前执行
    //如果返回true执行下一个拦截器/控制器，即放行
    //如果返回false就不执行下一个拦截器/控制器，即不放行
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("------------处理前------------");
        return true;
    }

    //在控制器方法执行之后执行（没有发生异常时才会执行）
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("------------处理后------------");
    }

    //在dispatcherServlet处理后执行，做清理工作（无论是否发生异常都会执行）
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("------------清理------------");
    }
}
```

#### 配置bean
```xml
<!--关于拦截器的配置-->

<!-- 可以配置多个拦截器，
执行的顺序参照配置的顺序
-->
<mvc:interceptors>
    <!-- 配置指定1个拦截器 -->
    <mvc:interceptor>
        <!-- 
        配置拦截指定控制器方法路径
        path="拦截路径"
        精确匹配：path="/dept/save" 拦截dept/save控制器方法
        模糊匹配：path="/dept/**" 拦截dept开头的所有控制器方法, 注意“**”代表任意长度字符，不支持一个"*" 
        -->

        <!--/admin/* 拦截的是/admin/add等等这种，/admin/add/user不会被拦截-->
        <!--/admin/** 拦截的是/admin/下的所有-->
        <mvc:mapping path="/**"/>
        <!-- 通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求 -->
        <mvc:exclude-mapping path="/test"/>
        <!--bean配置的就是拦截器-->
        <bean class="com.kuang.interceptor.MyInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 运行流程
![springmvc+20210802091610](https://i.loli.net/2021/08/02/ya1xpkf5eG93UlA.png)


## 静态资源访问
### 问题描述
springMVC中，项目配置的DispatcherServlet（这个Servlet不能处理静态资源）的拦截请求路径是“/”，覆盖了tomcat提供的默认servlet（在$CATALINA_HOME/conf/web.xml中）。该默认servlet用于处理静态资源，它被覆盖就会导致静态资源访问不了。


![springmvc+20210801235435](https://i.loli.net/2021/08/01/UgGz85p9TVSckKL.png)
### 处理静态资源方案1
在项目的web.xml中将静态资源的处理交换给全局配置的默认的Servlet。
```xml
<!--静态资源处理方案1-->
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.html</url-pattern>
    <url-pattern>*.js</url-pattern>
    <url-pattern>*.css</url-pattern>
    <url-pattern>*.jpg</url-pattern>
</servlet-mapping>
```
### 处理静态资源方案2
SpringMVC提供了\<mvc:resources/>标签，用于解决该问题。

\<mvc:resources/> 让Spring MVC框架自己处理静态资源，传统Web容器的静态资源只能放在Web容器的根路径下，而<mvc:resources/>允许静态资源放在任何地方，如：WEB-INF目录下、类路径下等。它有如下两个属性：
* location：指定静态资源的位置，可以使用诸如"classpath:"等的资源前缀指定资源位置。可以同时指定多个路径使用逗号分隔，路径要以/结束
* mapping：映射地址(即访问地址)，以/**结尾，它表示映射目录下所有的URL，包括子孙路径的资源

#### springMVC.xml
```xml
<!--
    解决静态资源直接访问的方案2：通过标签<mvc:resources>解决
    注意点：
        1.mapping属性：配置映射的任何静态资源，注意必须是2个*号
        2.location：配置直接访问的资源路径
            第一个"/"，代表直接访问webapp目录下的静态资源文件
            第二个"classpath:static/"，代表直接访问resources/static目录下的静态资源文件
            第三个“/WEB-INF/”,代表访问webapp/WEB-INF/目录下的静态资源文件
            多个路径之间使用逗号隔开，路径必须以“/”结束
-->
<mvc:resources mapping="/**" location="/,classpath:static/,/WEB-INF/">
```
### 处理静态资源方案3
在spring-mvc.xml中配置\<mvc:default-servlet-handler />后，会在Spring MVC容器中定义一个
org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler
它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。
![springmvc+20210802001303](https://i.loli.net/2021/08/02/nVS1KCajOB4xckv.png)

#### springMVC.xml
```xml
<!--
    解决静态资源直接访问的方案3：通过标签<mvc:default-servlet-handler/>交给tomcat默认的DefaultServlet处理
    注意：
        1.所有的静态资源都必须放在webapp目录下
        2.不同的服务器，默认的Servlet名字都不一样。当前默认名字不是default的时候，必须通过default-servlet-name属性指明具体的名字
            例如：webLogic服务器，就需需要default-servlet-name="FileServlet"
-->
<mvc:default-servlet-handler default-servlet-name="default">
```


## 异常处理
在三层架构中，如果Dao持久层有异常可以抛出到调用方(Service)，Service有异常可以抛出到Controller，但是Controller有异常，不建议抛出到客户端(用户)，因为用户看不懂异常，无法进行处理会导致用户体验非常差。

异常处理的原则：出现异常要给用户友好提示，自动跳转到友好提示页面。

### 自定义异常过滤器
```java
@WebFilter("/*")
public class ExceptionFilter implements Filter {
   public void doFilter(ServletRequest request, ServletResponse response,
FilterChain chain){
       try{
           chain.doFilter(request, response);
       }catch(Exception e){
           // 异常处理，跳转到友好页面
       }
   }
}
```　

### 基于配置的异常处理
只需要写一个类实现HandlerExceptionResolver接口，就可以捕获controller中的异常：
```java
@Component
public class CustomException implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {

        //处理异常目标：让开发人员看到异常信息 和 让用户看到友好信息
        //参数中Exception e 就是捕获到的异常对象
        e.printStackTrace(); //以后专业的做法，写入日志文件

        //实例ModelAndView
        ModelAndView mv = new ModelAndView();

        //友好信息写入到请求域中
        mv.addObject("errorMsg","服务器忙，请明天再来！"+e.getMessage());

        //返回视图页面：/pages/error.jsp
        mv.setViewName("error");
        return mv;
    }
}
```

在web.xml配置友好页面中
```xml
<!--错误页面配置-->
<error-page>
   <error-code>404</error-code>
   <location>/pages/404.jsp</location>
</error-page>
<error-page>
   <error-code>400</error-code>
   <location>/pages/400.jsp</location>
</error-page>
```

### 基于注解的异常处理
```java
//@ControllerAdvice将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {

    //@ExceptionHandler用于设置所标识方法处理的异常
    @ExceptionHandler(ArithmeticException.class)
    //ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        model.addAttribute("ex", ex);
        return "error";
    }

}
```


## 参考
* https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483970&idx=1&sn=352e571ee88957ce391e972344e2a3d7&scene=19
* https://www.cnblogs.com/xiaoxi/p/6164383.html
* http://www.51gjie.com/javaweb/911.html
* https://www.jianshu.com/p/8a20c547e245