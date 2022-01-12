# JavaConfig
Spring JavaConfig is a product of the Spring community that provides a pure-Java approach to configuring the Spring IoC Container. 

Spring JavaConfig 是 Spring 社区的一个产品，它提供了纯Java的方式去配置 Spring IOC 容器。

## 简介
JavaConfig 原来是 Spring 的一个子项目，它通过 Java 类的方式提供 Bean 的定义信息，在 Spring4 的版本，JavaConfig 已正式成为 Spring4 的核心功能。

## 核心注解

### @Configuration
@Configuration放在类上，表示这是一个配置类，等价于XML配置文件中的\<beans>元素，即XML配置文件的根元素。一个应用可以包含一到多个被@Configuration注解的类。

### @Bean
1. @Bean注解用在方法上，它直接等价于XML配置文件中的\<bean>元素。
2. 自动将方法的返回值加入到Spring容器中，方法名就是放在容器中的id。
3. 如果想指定与方法名不同的id，则使用name属性指定名字，相当于指定id(name属性的别名是value)。
4. 如果方法有参数，则参数传入的对象从容器中自动按类型匹配的方式去找；或者直接调用配置类中其他方法获取所需的参数。

#### @ComponentScan
放在类上，指定要扫描的基包。
| @ComponentScan的属性 | 作用                                         |
| -------------------- | -------------------------------------------- |
| basePackages         | 参数是一个字符数组，指定一个或多个基包的名字 |
| value                | 同上                                         |

#### @PropertySource
读取Java的属性文件(.properties)，value[]属性：指定一个或多个属性文件的名字。  
@PropertySource可以不用写classpath，因为注解默认从类路径下加载

#### @ImportResource
导入Spring的配置文件（beans.xml），让配置文件里面的内容生效

#### @Import
@Import是一个功能强大的注解，它通过指定一个或多个类对象，实现了如下的功能：
* 导入其他的配置类：在已有的Bean上，使用@Import注解，导入一个不在@ComponentScan扫描范围内的配置类
* 直接/间接将其他类导入到容器中
  * 直接指定其他的类的：在已有的Bean上，使用@Import注解，导入一个普通类。
  * 指定实现ImportSelector的类：在已有的Bean上，使用@Import注解，导入一个ImportSelector的实现类。该实现类在重写的selectImports方法中，返回的类全路径（以String[]的形式），返回值中的所有的类都会变成Bean。
  * 指定实现ImportBeanDefinitionRegistrar的类：已有的Bean上，使用@Import注解，导入一个ImportBeanDefinitionRegistrar的实现类，在重写的方法中注册BeanDefinition到Spring容器。



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