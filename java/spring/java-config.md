# JavaConfig
Spring JavaConfig is a product of the Spring community that provides a pure-Java approach to configuring the Spring IoC Container. 

Spring JavaConfig 是 Spring 社区的一个产品，它提供了纯Java的方式去配置 Spring IOC 容器。

## 简介
JavaConfig 原来是 Spring 的一个子项目，它通过 Java 类的方式提供 Bean 的定义信息，在 Spring4 的版本，JavaConfig 已正式成为 Spring4 的核心功能。

## 注解

### @Configuration
@Configuration放在类上，表示这是一个配置类，等价于XML配置文件中的\<beans>元素，即XML配置文件的根元素。一个应用可以包含一到多个被@Configuration注解的类。

### @Bean
1. @Bean注解用在方法上，它直接等价于XML配置文件中的\<bean>元素。
2. 自动将方法的返回值加入到Spring容器中，方法名就是放在容器中的id。
3. 如果想指定与方法名不同的id，则使用name属性指定名字，相当于指定id(name属性的别名是value)。
4. 如果方法有参数，则参数传入的对象从容器中自动按类型匹配的方式去找；或者直接调用配置类中其他方法获取所需的参数。

### @ComponentScan
放在类上，指定要扫描的基包。
| @ComponentScan的属性 | 作用                                         |
| -------------------- | -------------------------------------------- |
| basePackages         | 参数是一个字符数组，指定一个或多个基包的名字 |
| value                | 同上                                         |

### @PropertySource
读取Java的属性文件(.properties)，value[]属性：指定一个或多个属性文件的名字。  
@PropertySource可以不用写classpath，因为注解默认从类路径下加载

### @ImportResource
导入Spring的配置文件（beans.xml），让配置文件里面的内容生效

### @Import
@Import是一个功能强大的注解，它通过指定一个或多个类对象，实现了如下的功能：
* 导入其他的配置类：在已有的Bean上，使用@Import注解，导入一个不在@ComponentScan扫描范围内的配置类
* 直接/间接将其他类导入到容器中
  * 直接指定其他的类的：在已有的Bean上，使用@Import注解，导入一个普通类。
  * 指定实现ImportSelector的类：在已有的Bean上，使用@Import注解，导入一个ImportSelector的实现类。该实现类在重写的selectImports方法中，返回的类全路径（以String[]的形式），返回值中的所有的类都会变成Bean。
  * 指定实现ImportBeanDefinitionRegistrar的类：已有的Bean上，使用@Import注解，导入一个ImportBeanDefinitionRegistrar的实现类，在重写的方法中注册BeanDefinition到Spring容器。

### @ConfigurationProperties

### @EnableConfigurationProperties




### 示例
#### 实体类
```java
@Component  //将这个类标注为Spring的一个组件，放到容器中！
public class Dog {
   public String name = "dog";
}
```

#### 配置类
```java
@Configuration  //代表这是一个配置类
@PropertySource("druid.properties")  // 加载配置文件
@Import(OtherConfig.class)  // 导入其他配置类
@ComponentScan("com.itheima")  // 扫描基包的中类，以支持注解配置功能
public class MyConfig {

    @Value("${jdbc.username}")  // 使用配置文件中的属性
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean //通过方法注册一个bean，这里的返回值就Bean的类型，方法名就是bean的id！
    public Dog dog(){
       return new Dog();
   }

    @Bean
    public DataSource createDataSource() {
        /* 省略部分代码 */
        return ds;
    }

    @Bean // 方法有参数ds，按类型DataSource匹配的方式从容器中去查找
    public JdbcTemplate createJdbcTemplate(DataSource ds) {
        return new JdbcTemplate(ds);
    }
}
```

#### 测试类
```java
@Test
public void test(){
   ApplicationContext applicationContext =
           new AnnotationConfigApplicationContext(MyConfig.class);
   Dog dog = (Dog) applicationContext.getBean("dog");
   System.out.println(dog.name);
}
```

## JavaConfig启动
在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器。

Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。

Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了 AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

### WebInit
```java
public class WebInit
    extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
    * 指定spring的配置类 * @return
    */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { SpringConfig.class };
    }

    /**
    * 指定SpringMVC的配置类 * @return
    */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { WebConfig.class };
    }

    /**
    * 指定DispatcherServlet的映射规则，即url-pattern * @return
    */
    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

    /**
    * 添加过滤器 * @return */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);

        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();

        return new Filter[] { encodingFilter, hiddenHttpMethodFilter };
    }
}
```

### SpringConfig

```java
@Configuration
public class SpringConfig { //ssm整合之后，spring的配置信息写在此类中
}
```

### WebConfig

```java
@Configuration
//扫描组件
@ComponentScan("com.atguigu.mvc.controller")
//开启MVC注解驱动
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    //使用默认的servlet处理静态资源
    @Override
    public void configureDefaultServletHandling(
        DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //配置文件上传解析器
    @Bean
    public CommonsMultipartResolver multipartResolver() {
        return new CommonsMultipartResolver();
    }

    //配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();
        registry.addInterceptor(firstInterceptor).addPathPatterns("/**");
    }

    //配置视图控制
    /*@Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }*/

    //配置异常映射
    /*@Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");
        //设置异常映射
        exceptionResolver.setExceptionMappings(prop);
        //设置共享异常信息的键
        exceptionResolver.setExceptionAttribute("ex");
        resolvers.add(exceptionResolver);
    }*/

    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();

        // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);

        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(
        ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);

        return templateEngine;
    }

    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);

        return viewResolver;
    }
}
```

### 测试

```java
@RequestMapping("/")
public String index(){
    return "index";
}
```