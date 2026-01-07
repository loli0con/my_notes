# 注解 的 配置方式
从 Java 5 开始，Java 增加了对注解（Annotation）的支持，它是代码中的一种特殊标记，可以在编译、类加载和运行时被读取，执行相应的处理。开发人员可以通过注解在不改变原有代码和逻辑的情况下，在源代码中嵌入补充信息。Spring 从 2.5 版本开始提供了对注解技术的全面支持，我们可以使用注解来实现自动装配，简化 Spring 的 XML 配置。

Spring 通过注解实现自动装配的步骤如下：
1. 引入依赖
2. 开启组件扫描
   1. [使用XML配置](#开启组件扫描)
   2. [使用@ComponentScan](#componentscan)
3. 使用注解定义Bean
4. 依赖注入


## 开启组件扫描
Spring 默认不使用注解装配 Bean，因此我们需要在 Spring 的 XML 配置中，通过 `<context:component-scan>` 元素开启 Spring Beans的自动扫描功能。开启此功能后，Spring 会自动从扫描指定的包（base-package 属性设置）及其子包下的所有类，如果类上使用了 @Component 注解，就将该类装配到容器中。

需要在xml配置文件中做如下配置：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 最基本的扫描方式：扫描哪个包下所有的类，指定基包的名字（可以使用逗号分隔多个基包）-->
    <context:component-scan base-package="com.itheima"/>
    
    <!-- 指定要排除的组件 -->
    <context:component-scan base-package="com.atguigu.spring6">
        <!-- context:exclude-filter标签：指定排除规则 -->
        <!-- 
            type：设置排除或包含的依据
            type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
            type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
        -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:exclude-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>
    </context:component-scan>

    <!-- 仅扫描指定组件 -->
    <context:component-scan base-package="com.atguigu" use-default-filters="false">
        <!-- context:include-filter标签：指定在原有扫描规则的基础上追加的规则 -->
        <!-- use-default-filters属性：取值false表示关闭默认扫描规则 -->
        <!-- 此时必须设置use-default-filters="false"，因为默认规则即扫描指定包下所有类 -->
        <!-- 
            type：设置排除或包含的依据
            type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
            type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
        -->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:include-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>
    </context:component-scan>
</beans>
```



## Bean创建

### @Component - 代替<bean>标签的id和class属性
Spring 提供了以下多个注解，这些注解可以直接标注在 Java 类上，将它们定义成 Spring Bean。

默认是类名首字母小写做为beanName，也可以使用value属性来指定*该类的名字*（相当于配置文件中`id="该类的名字" class="当前注解的类的全类名"/>`）。

#### @Component衍生注解（子注解）
|注解|说明|
|---|---|
|@Controller|放在控制器上，在Web层类上使用|
|@Service|放在业务对象上，在Service层类上使用|
|@Repository|放在持久层，在Dao层类上使用|

以上三个注解的功能一样，只是语义上区别。

### @Scope - 代替<bean>标签的scope属性
在类上或使用了@Bean标注的方法上，标注Bean的作用范围，取值为singleton或prototype。

### @Lazy - 代替<bean>标签的lazy-init属性
在类上或使用了@Bean标注的方法上，标注Bean是否延迟加载，在默认的情况(false)下，容器加载对象就加载了。如果为true表示延迟加载。

### @PostConstruct - 代替<bean>标签的init-method属性
在方法上使用，标注Bean的实例化后执行的方法。Spring只根据Annotation查找无参数方法，对方法名不作要求。

### @PreDestroy - 代替<bean>标签的destory-method属性
在方法上使用，标注Bean销毁前执行的方法。只适用于单例对象。Spring只根据Annotation查找无参数方法，对方法名不作要求。



## 依赖注入

### @Value
用在字段或方法上，用于注入普通数据。也可以[读取配置文件中值注入](https://www.cnblogs.com/binghe001/p/13216798.html)——结合@PropertySource等注解使用。
```java
public Class User{
    @Value("${jdbc.username:root}")
    private username;

    @Value("root")
    public void setUsername(String username){
        System.out.println(username); 
    }

    @Value("true")
    private boolean sex;

    @Value("2011/11/11")
    private Date birthday;

    @Value("file:/path/to/logo.txt")
    private Resource resource;
}
```

### @Autowired
Autowired注解放在属性上、构造方法上、构造方法的参数上、setter方法上，表示将*相应的值*注入该属性（从容器中寻找合适的被依赖资源）。*相应的值*：
* 按类型匹配的方式从容器中去查找对应的值注入
* 如果有多个匹配的类型，按名字匹配的方式注入
* 如果找不到匹配的名字，就会抛出异常

@Autowired的属性required：默认为true，表示这个属性是必需的；如果为false，则匹配失败时，不抛出异常，将属性值为null。

当带参数的构造方法只有一个，@Autowired注解可以省略。

### @Qualifier
@Autowired是根据类型自动装配的，加上@Qualifier则可以根据byName的方式自动装配：
* 位置: 放在成员变量或成员方法上
* 作用: 按名字匹配的方式注入
* 属性: value指定要注入的名字

必须与@Autowired配置使用，不能单独使用。

### @Resource
Resource注解放在成员变量和成员方法上，表示从容器中寻找值并进行注入：
* 如有指定的name属性，按该属性进行byName方式查找装配；
* 否则按照默认的byName方式进行装配；
* 如果以上都不成功，则按byType的方式自动装配。
* 都不成功，则报异常。



## @Bean - 工厂方法
非自定义Bean不能像自定义Bean一样使用@Component进行管理，非自定义Bean要通过工厂的方式进行实例化，使用@Bean标注方法即可，@Bean的属性为beanName，如不指定为当前工厂方法名称。Spring对标记为@Bean的方法只调用一次，因此返回的Bean仍然是单例。可以用@Bean("*名字*")指定别名，也可以用@Bean+@Qualifier("*名字*")指定别名。

工厂方法所在类必须要被Spring管理。

如果@Bean工厂方法需要参数的话，则有如下几种注入方式：
* 使用@Autowired根据类型自动进行Bean的匹配，@Autowired可以省略；
* 使用@Qualifier根据名称进行Bean的匹配；
* 使用@Value根据名称进行普通数据类型匹配。

```Java
//将方法返回值Bean实例以@Bean注解指定的名称存储到Spring容器中
@Bean("myDataSource")
public DataSource dataSource(){
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis");
    dataSource.setUsername("root");
    dataSource.setPassword("root");
    return dataSource;
}

@Bean
@Autowired //根据类型匹配参数
public Object objectDemo01(UserDao userDao){
    System.out.println(userDao);
    return new Object();
}
@Bean
public Object objectDemo02(
    @Qualifier("userDao") UserDao userDao,
    @Value("${jdbc.username}") String username){
    System.out.println(userDao);
    System.out.println(username);
    return new Object();
}
```



## 配置类注解

### @Configuration
从Spring3.0，@Configuration用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

### @ComponentScan
默认自动扫描当前类所在的包及其子包，把所有标注为@Component的Bean都找出来，生成对应的BeanDefinition并进行注册。可以通过指定一个或多个包名：扫描指定包及其子包。

### @PropertySource
Spring容器还提供了一个更简单的@PropertySource来家在外部properties资源配置。

先使用@PropertySource读取配置文件，然后通过@Value以`${key:defaultValue}`的形式注入，可以极大地简化读取配置的麻烦。

### @Import
@Import用于加载其他配置类



## 其他注解

### @Primary
@Primary注解用于标注相同类型的Bean优先被使用权，与@Component和@Bean一起使用，标注该Bean的优先级更高，则在通过类型获取Bean或通过@Autowired根据类型进行注入时，会选用优先级更高的。

### @Profile
注解@Profile标注在类或方法上，标注当前产生的Bean从属于哪个环境，只有激活了当前环境，被标注的Bean才能被注册到Spring容器里，不指定环境的Bean，任何环境下都能注册到Spring容器里。