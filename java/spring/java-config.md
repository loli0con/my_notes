# JavaConfig
Spring JavaConfig is a product of the Spring community that provides a pure-Java approach to configuring the Spring IoC Container. 

Spring JavaConfig 是 Spring 社区的一个产品，它提供了纯Java的方式去配置 Spring IOC 容器。

## 简介
JavaConfig 原来是 Spring 的一个子项目，它通过 Java 类的方式提供 Bean 的定义信息，在 Spring4 的版本，JavaConfig 已正式成为 Spring4 的核心功能。

## 注解
使用JavaConfig的时候，会使用到大量的注解，即以注解的形式进行配置。

参考[Spring注解驱动开发系列教程](https://liayun.blog.csdn.net/article/details/115053350)，可以获得更详细的示例。

### @Configuration
@Configuration放在类上，表示这是一个配置类，等价于XML配置文件中的\<beans>元素，即XML配置文件的根元素。一个应用可以包含一到多个被@Configuration注解的类。

### @Bean
1. @Bean注解用在方法上，它直接等价于XML配置文件中的\<bean>元素。
2. 自动将方法的返回值加入到Spring容器中，方法名就是放在容器中的id。
3. 如果想指定与方法名不同的id，则使用name属性指定名字，相当于指定id(name属性的别名是value)。
4. 如果方法有参数，则参数传入的对象从容器中自动按类型匹配的方式去找；或者直接调用配置类中其他方法获取所需的参数。

@Bean注解的initMethod属性和destroyMethod属性可以指定bean的初始化方法和销毁方法：
* bean对象的初始化方法调用的时机：对象创建完成，如果对象中存在一些属性，并且这些属性也都赋好值之后，那么就会调用bean的初始化方法。对于单实例bean来说，在Spring容器创建完成后，Spring容器会自动调用bean的初始化方法；对于多实例bean来说，在每次获取bean对象的时候，调用bean的初始化方法。
* bean对象的销毁方法调用的时机：对于单实例bean来说，在容器关闭的时候，会调用bean的销毁方法；对于多实例bean来说，Spring容器不会管理这个bean，也就不会自动调用这个bean的销毁方法了。不过可以手动调用多实例bean的销毁方法。


### @Scope
@Scope注解用于设置组件的作用域

![java-config+20220204162636](https://raw.githubusercontent.com/loli0con/picgo/master/images/java-config%2B20220204162636.png%2B2022-02-04-16-26-37)

[自定义Scope](https://liayun.blog.csdn.net/article/details/110262802#t9)。

### @Lazy
懒加载/延迟加载：单实例bean是在Spring容器启动的时候加载的，添加@Lazy注解后就会延迟加载，在Spring容器启动的时候并不会加载，而是在第一次使用此bean的时候才会加载，但当你多次获取bean的时候并不会重复加载，只是在第一次获取的时候才会加载，这不是延迟加载的特性，而是单实例bean的特性。仅针对单实例bean生效。 

### @ComponentScan
放在类上，指定要扫描的基包。
| @ComponentScan的属性 | 作用                                         |
| -------------------- | -------------------------------------------- |
| basePackages         | 参数是一个字符数组，指定一个或多个基包的名字 |
| value                | 同上                                         |

可以使用excludeFilters()方法来指定扫描时排除哪些组件，也可以使用includeFilters()方法来指定扫描时只包含哪些组件。两个方法的返回值都是Filter\[]数组，[Filter](https://liayun.blog.csdn.net/article/details/110239421)存在于ComponentScan注解的内部存在。  
当使用includeFilters()方法指定只包含哪些组件时，需要禁用掉默认的过滤规则：`useDefaultFilters=false`。

@ComponentScan注解就是一个重复注解，可以在一个类上重复使用这个注解。

### @PropertySource
通过@PropertySource注解可以将properties配置文件中的key/value存储到Spring的Environment中，Environment接口提供了方法去读取配置文件中的值，参数是properties配置文件中定义的key值。也可以使用@Value注解用${}占位符为bean的属性注入值。

value[]属性：指定一个或多个属性文件的名字，例如`@PropertySource(value={"classpath:/person.properties"})`。@PropertySource可以不用写classpath，因为注解默认从类路径下加载。


### @ImportResource
导入Spring的配置文件（beans.xml），让配置文件里面的内容生效

### @Import
@Import是一个功能强大的注解，它通过指定一个或多个类对象，实现了如下的功能：
* 导入其他的配置类：在已有的Bean上，使用@Import注解，导入一个不在@ComponentScan扫描范围内的配置类
* 直接/间接将其他类导入到容器中
  * 直接指定其他的类：在已有的Bean上，使用@Import注解，导入一个或多个普通类。
  * 指定实现ImportSelector的类：在已有的Bean上，使用@Import注解，导入一个ImportSelector的实现类。该实现类在重写的selectImports方法中，返回的类全路径（以String[]的形式），返回值中的所有的类都会变成Bean。
  * 指定实现ImportBeanDefinitionRegistrar的类：已有的Bean上，使用@Import注解，导入一个ImportBeanDefinitionRegistrar的实现类，在重写的方法中注册BeanDefinition到Spring容器。

### @ConfigurationProperties

### @EnableConfigurationProperties

### @Conditional
@Conditional注解可以按照一定的条件进行判断，满足条件向容器中注册bean，不满足条件就不向容器中注册bean。@Conditional注解可以放到类上，也可以放到方法上。

@Conditional注解中的value参数，对应一个Condition类型或者其子类型的Class对象数组。Condition类型是一个接口，使用@Conditional注解时，传入Condition接口的实现，实现该接口内的matches抽象方法，以该方法返回的返回值(布尔值)作为判断依据来决定是否进行注入。

![java-config+20220204170352](https://raw.githubusercontent.com/loli0con/picgo/master/images/java-config%2B20220204170352.png%2B2022-02-04-17-03-54)


### @PostConstruct
@PostConstruct注解是JSR-250规范里面定义的一个注解。

@PostConstruct注解被用来修饰一个非静态的void()方法。被@PostConstruct注解修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。被@PostConstruct注解修饰的方法通常在构造函数之后，init()方法之前执行。

执行顺序如下：Constructor（构造方法）→@Autowired（依赖注入）→@PostConstruct（注释的方法）

### @PreDestroy
@PreDestroy注解是JSR-250规范里面定义的一个注解。
被@PreDestroy注解修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy注解修饰的方法会在destroy()方法之后，Servlet被彻底卸载之前执行。

执行顺序如下所示：调用destroy()方法→@PreDestroy→destroy()方法→bean销毁

### @Value
Spring中的@Value注解可以为bean中的属性赋值，它可以标注在字段、方法、参数以及注解上，而且在程序运行期间生效。

#### 用法 - 不通过配置文件注入属性的情况
```java
@Value("李阿昀")
private String name; // 注入普通字符串

@Value("#{systemProperties['os.name']}")
private String systemPropertiesName; // 注入操作系统属性

@Value("#{ T(java.lang.Math).random() * 100.0 }")
private double randomNumber; //注入SpEL表达式结果

@Value("#{person.name}")
private String username; // 注入其他bean中属性的值，即注入person对象的name属性中的值

@Value("classpath:/config.properties")
private Resource resourceFile; // 注入文件资源

@Value("http://www.baidu.com")
private Resource url; // 注入URL资源
```

#### 用法 - 通过配置文件注入属性的情况
```java
@PropertySource(value={"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {

	@Bean
	public Person person() {
		return new Person();
	}
	
}

class Person {
	
	@Value("李阿昀")
	private String name;
	@Value("#{20-2}")
	private Integer age;
	
	@Value("${person.nickName:meimeixia}") // meimeixia为默认值
	private String nickName;
}
```

#### #{···}和${···}的区别
两者的内容都必须符合SpEL表达式。

区别：
* `#{···}`：用于执行SpEl表达式，并将内容赋值给属性
* `${···}`：主要用于加载外部属性文件中的值
* `${···}`和`#{···}`可以混合使用，但是必须`#{}`在外面，`${}`在里面


### @Autowired
@Autowired注解可以对类成员变量、构造方法、实例方法和参数进行标注，完成自动装配的工作。

@Autowired注解默认是优先按照类型去容器中找对应的组件，若找到则就赋值；如果找到多个相同类型的组件，那么是将属性名称作为组件的id，到IOC容器中进行查找。

无论@Autowired注解是标注在字段上、实例方法上、构造方法上还是参数上，参数位置的组件都是从IOC容器中获取。

使用@Autowired注解标注在构造方法上时，组件中如果只有一个有参构造方法，那么这个有参构造方法上的@Autowired注解可以省略，方法的参数将自动从IOC容器中获取。

### @Qualifier
@Autowired是根据类型进行自动装配的，如果需要按名称进行装配，那么就需要配合@Qualifier注解来使用了。

### @Primary
在Spring中使用注解时，常常会使用到@Autowired这个注解，它默认是根据类型Type来自动注入的。但有些特殊情况，对同一个接口而言，可能会有几种不同的实现类，而在默认只会采取其中一种实现的情况下，就可以使用@Primary注解来标注优先使用哪一个实现类。

### @Resource
@Resource注解是Java规范里面的，它是JSR250规范里面定义的一个注解。该注解默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，那么默认取字段名将其作为组件的名称在IOC容器中进行查找，如果注解写在setter方法上，那么默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的一点是，如果name属性一旦指定，那么就只会按照名称进行装配。

@Resource注解和@Autowired注解的功能是一样的，都能实现自动装配，只不过@Resource注解默认是按照组件名称（即属性的名称）进行装配的。虽然@Resource注解具备自动装配这一功能，但是它是不支持@Primary注解优先注入的功能的，而且也不能像@Autowired注解一样能添加required=false属性。

### @Inject
@Inject注解也是Java规范里面的，它是JSR330规范里面定义的一个注解。该注解默认是根据参数名去寻找bean注入，支持Spring的@Primary注解优先注入，@Inject注解还可以增加@Named注解指定要注入的bean。

要想使用@Inject注解，需要导入javax.inject这个包（依赖包）。

@Inject注解和@Autowired注解的功能是一样的，都能实现自动装配，而且它俩都支持@Primary注解优先注入的功能。只不过，@Inject注解不能像@Autowired注解一样能添加required=false属性，因为它里面没啥属性。

### @Profile
在容器中如果存在同一类型的多个组件，那么可以使用@Profile注解标识要获取的是哪一个bean。也可以说@Profile注解是Spring为我们提供的可以根据当前环境，动态地激活和切换一系列组件的功能。这个功能在不同的环境使用不同的变量的情景下特别有用，例如，开发环境、测试环境、生产环境使用不同的数据源，在不改变代码的情况下，可以使用这个注解来动态地切换要连接的数据库。

@Profile注解不仅可以标注在方法上，也可以标注在配置类上。如果@Profile注解标注在配置类上，那么只有是在指定的环境的时候，整个配置类里面的所有配置才会生效。如果一个bean上没有使用@Profile注解进行标注，那么这个bean在任何环境下都会被注册到IOC容器中（前提是在配置生效的情况下）。




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


## 接口
除了使用注解进行配置，还有其他的方式实现配置。

### FactoryBean
Spring提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，可以通过实现该接口定制实例化bean的逻辑。

![java-config+20220204173749](https://raw.githubusercontent.com/loli0con/picgo/master/images/java-config%2B20220204173749.png%2B2022-02-04-17-37-50)

方法说明：
* T getObject()：返回由FactoryBean创建的bean实例，如果isSingleton()返回true，那么该实例会放到Spring容器中单实例缓存池中
* boolean isSingleton()：返回由FactoryBean创建的bean实例的作用域是singleton还是prototype
* Class getObjectType()：返回FactoryBean创建的bean实例的类型

注入过程：
```java
// 某个JavaConfig中
@Bean
public MyFactoryBean myFactoryBean() {
    return new MyFactoryBean(); // FactoryBean接口实现类
}
```

### InitializingBean
Spring中提供了一个InitializingBean接口，该接口为bean提供了属性初始化后的处理方法，它只包括afterPropertiesSet方法，凡是继承该接口的类，在bean的属性初始化后都会执行该方法。

Spring为bean提供了两种初始化的方式，第一种方式是实现InitializingBean接口（也就是要实现该接口中的afterPropertiesSet方法），第二种方式是在配置文件或@Bean注解中通过init-method来指定，这两种方式可以同时使用，同时使用先调用afterPropertiesSet方法，后执行init-method指定的方法。

### DisposableBean
实现org.springframework.beans.factory.DisposableBean接口的bean在销毁前，Spring将会调用DisposableBean接口的destroy()方法。也就是说可以实现DisposableBean这个接口来定义销毁的逻辑。

### BeanPostProcessor
BeanPostProcessor是一个接口，其中有两个方法，即postProcessBeforeInitialization和postProcessAfterInitialization这两个方法，这两个方法分别是在Spring容器中的bean初始化前后执行，所以Spring容器中的每一个bean对象初始化前后，都会执行BeanPostProcessor接口的实现类中的这两个方法。

postProcessBeforeInitialization方法会在bean实例化和属性设置之后，自定义初始化方法之前被调用，而postProcessAfterInitialization方法会在自定义初始化方法之后被调用。当容器中存在多个BeanPostProcessor的实现类时，会按照它们在容器中注册的顺序执行。对于自定义的BeanPostProcessor实现类，还可以让其实现Ordered接口自定义排序。

Spring提供了BeanPostProcessor接口的很多实现类，例如AutowiredAnnotationBeanPostProcessor用于@Autowired注解的实现，AnnotationAwareAspectJAutoProxyCreator用于Spring AOP的动态代理等等。

