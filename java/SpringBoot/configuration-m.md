# 配置管理

## 2.1 配置类

### 2.1.1 自定义配置类
`@SpringBootConfiguration`是Spring Boot应用的专用配置注解类，它组合了Spring框架的原生`@Configuration`注解，它俩是等效的。

### 2.1.2 导入配置

#### 1. 导入配置类
如果配置类的路径在类扫描路径下，直接用`@ComponentScan`注解扫描即可；如果配置类不在默认的类扫描路径下，需要用`@ComponentScan`注解指定要扫描的包目录。

`@Import`注解可以额外导入多个配置，具体的类型可以是以下几种：
* `@Configuration`
* `ImportSelector`
* `ImportBeanDefinitionRegister`
* 任何一个`component`组件

#### 2. 导入XML配置
Spring框架提供了一个`ImportResource`注解用于导入额外的XML配置文件。



## @PropertySource
自定义的配置文件并不会被自动加载，需要使用 Spring 提供的 @PropertySource 注解，去加载指定的配置文件。

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {
    String name() default "";

    String[] value();

    boolean ignoreResourceNotFound() default false;

    String encoding() default "";

    Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;
}
```
* name：指明此属性源的名称。如果省略，将根据属性源生成一个名称。
* value：要加载的属性文件的资源位置。支持 properties 和 XML 的属性文件格式，例如：`classpath:/cn/cxyxj/xx.properties`或`file:/path/xxx.xml`。注意：不允许使用资源位置通配符（例如 `**/*.properties`）
* ignoreResourceNotFound：指定的属性文件如果不存在，是否要忽略错误。默认为false，也就是属性文件不存在，则会抛出异常。
* encoding：指定资源文件的编码格式。不指定则使用文件默认的编码格式。
* factory：指定自定义`PropertySourceFactory`，默认使用 `DefaultPropertySourceFactory`。

### 加载yml
让@PropertySource注解支持yml文件格式需要使用到factory属性了，指定PropertySourceFactory进行加载。创建YmlPropertyResourceFactory类，并实现 PropertySourceFactory，重写createPropertySource方法。

```Java
import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.DefaultPropertySourceFactory;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.core.io.support.PropertySourceFactory;

import java.io.IOException;
import java.util.List;
import java.util.Optional;

/**
 * 在 @PropertySource 注解的 factory 属性指定 YmlPropertyResourceFactory 类，则可以支持读取 yml
 */
public class YmlPropertyResourceFactory implements PropertySourceFactory {

    private static String YML = ".yml";
    private static String YAML = ".yaml";
    /**
     *
     * @param name：@PropertySource 注解 name 的值
     * @param resource：资源
     */
    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        // 文件名称
        String filename = resource.getResource().getFilename();
        // 属性源的名称
        String resourceName = Optional.ofNullable(name).orElse(filename);
        if (filename.endsWith(YML) || filename.endsWith(YAML)) {
            List<PropertySource<?>> yamlSources = new YamlPropertySourceLoader().load(resourceName, resource.getResource());
            return yamlSources.get(0);
        } else {
            // 其他文件后缀
            return new DefaultPropertySourceFactory().createPropertySource(name, resource);
        }
    }
}
```



## 2.2 配置文件

### 2.2.1 application
Spring Boot应用的配置参数主要在application配置文件中，由StandardConfigDataLocationResolver类控制：
```Java
static final String CONFIG_NAME_PROPERTY = "spring.config.name";
static final String[] DEFAULT_CONFIG_NAMES = { "application" };
```
如果不想使用默认的application配置文件，则可以在命令行使用spring.config.name参数名指定其他的配置文件名称：
```sh
$ java -jar demo.jar --spring.config.name=app
```
默认的配置文件搜索路径为（由ConfigDataEnvironment类静态初始化块控制）：
* **optional:classpath:/**;**optional:classpath:/config/**。
* **optional:file:./**;**optional:file:./config/**;**optional:file:./config/*/**。

其中的**optional**前缀表示配置是可选的，如果指定的配置文件不存在，默认会报告异常，所以指定**optional**前缀就可以不用管配置文件是否存在。

默认的配置文件搜索路径可以通过spring.config.location命令行参数指定，既可以是目录（需要以“/”结束），也可以是具体文件。这个spring.config.location命令行参数会覆盖应用默认的配置文件，如果不想覆盖，可以使用spring.config.additional-location参数追加其他的配置文件。

### 2.2.3 配置文件类型
配置文件类型可以是以下两种：
* .properties：`key=value`格式
* .yml：`key: value`树状格式



## 2.3 配置绑定

### 2.3.1 Spring中的配置绑定
在Spring框架中，所有已加载到Spring环境中的配置都可以通过注入Environment环境Bean来获取：
```Java
@Autowired
private Environment env;

// 获取参数
String getProperty(String key);
```

使用`@Value("${property}")`注解在类成员变量上绑定配置参数是最原始的做法，一般不推荐这样做，这样注入会让代码太冗余和不优雅。

### 2.3.2 参数绑定
通过Java Bean提供的setter方法进行配置参数与Java Bean字段的绑定：
1. 一般参数类要以XxxProperties命名
2. 通过`@ConfigurationProperties`注解将配置参数映射到一个Java Bean上，该注解中的prefix或者value参数用于指定要映射的参数前缀，前缀格式为英文小写，多个前缀以“-”分隔
3. 在启动类上添加`@EnableconfigurationProperties`注解，用于指定要启用的`@ConfigurationProperties`配置参数类

`@ConfigurationProperties`注解配置参数绑定总结：
* 支持按配置参数的前缀进行绑定，前缀一样的配置将被绑定到同一个类上
* 支持配置参数使用默认值
* 支持松散绑定（**xx1-xx2-xx3** -> **xx1Xx2Xx3**），这里的**连接符-**也可以是**下划线_**，还可以是**XX_XX_XX的大写格式**
* 支持Java集合绑定
* 支持嵌套类
* 支持主要的配置途径，比如.yml文件、.properties文件、环境变量参数等
* 需要搭配`@EnableconfigurationProperties`注解共同使用

### 2.3.3 构造器绑定
通过`@ConfigurationProperties`注解指定要绑定的配置参数的前缀，再使用`@ConstructorBinding`注解指定到要绑定的构造器方法上。该构造器方法参数可以使用`@DefualtValue`给形参添加默认值，也可以用`@DateTimeFormat`指定形参的日期格式。也要在启动类`@EnableconfigurationProperties`注解中启用该配置参数类。

### 2.3.4 Bean配置绑定
当需要将参数绑定到一个第三方类上时（无法修改他人代码），可以配合使用`@Bean`和`@ConfigurationProperties`。该方式只是把`@ConfigurationProperties`注解从参数类中移到了@Bean方法之上，此时不需要使用`@EnableconfigurationProperties`注解来使`@ConfigurationProperties`配置类生效，直接注入就能使用。

这种方式只能绑定简单的数据类型，像Date这种数据类型就没有了应对的解决方案。

### 2.3.5 参数类扫描
Spring Boot提供了一种参数类扫描方式：在启动类上使用`@ConfigurationPropertiesScan`注解，就可以扫描所有包目录下的参数类，如果要扫描指定的包目录，则在注解上basePackages参数重指定具体的包即可。

### 2.3.6 配置验证
使用`@ConfigurationProperties`注解还能有效地进行参数验证，Spring Boot支持JSR-303 javax.validation规范注解。



## 2.4 外部化配置

### 2.4.1 配置源
Spring Boot还可以将配置外部化，即除了把配置放在应用配置文件中，还可以使用各种外部配置源，主要包含以下几种：
* properties文件
* YAML文件
* 环境变量
* 命令行参数

### 2.4.2 配置优先级
Spring Boot使用了一个非常特殊的PropertySource顺序，该顺序可以按优先级自动覆盖参数值，如果多个配置源的参数值相同，则优先级高的参数值会覆盖优先级低的参数值。配置源优先级从低到高如下：
1. 默认参数（SpringApplication.setDefaultProperties）
2. 使用@PropertySource注解绑定的配置
3. 应用配置文件（application）中的参数，多个应用配置文件（application）的优先级从低到高如下：
   1. （jar包内）应用配置文件
   2. （jar包内）指定了profile的配置文件，如application-dev.properties
   3. （jar包外）应用配置文件
   4. （jar包外）指定了profile的配置文件，如application-dev.properties
4. 配置了random.*随机数的参数
5. 系统环境变量
6. Java System Properties
7. java:comp/env的JNDI参数
8. ServletContext初始化参数
9.  ServletConfig初始化参数
10. 来自SPRING_APPLICATION_JSON的参数
11. 命令行参数
12. 单元测试上的参数
13. 使用@TestPropertySource注解绑定的配置
14. Devtool全局设置参数（来自$HOME/.config/spring-boot）

建议在整个应用中统一使用一种格式，但如果同一位置同时有.properties和.yml两种格式的配置文件，则Spring Boot会优先使用.properties配置文件。

### 2.4.3 命令行参数
命令行参数指的是使用`java -jar --key=value`命令启动应用时指定的参数，即以“--”开头的参数，比如：
```sh
java -jar demo.jar --server.port=9090
```

默认情况下，Spring Boot会将所有命令行参数转换并添加到Spring环境中，如果不想添加到Spring环境中，则可以将其禁用：
```Java
public static void main(String[] args){
    SpringApplication.setAddCommandLineProperties(false);
    SpringApplication.run(Application.class, args);
}
```



### 2.5 导入配置
可以使用spring.config.import参数来指定要导入的配置文件的路径，比如下面指定了要导入一个app.yml配置文件：
```yml
spring:
  config:
    import:
      - optional:classpath:/config/app.yml
```

导入其他位置的配置文件的优先级要高于触发导入配置文件的优先级：即a导入b，b的优先级高于a。



### 2.6 随机值配置
SpringBoot提供的RandomValuePropertySource类可用于注入随机值，它可以生成整数（int）、长整数（long）、UUID及字符串。
* 整数：${random.int}
* 长整数：${random.long}
* 整数范围：${random.int[0,100]}
* 长整数范围：${random.long[0,100]}
* UUID：${random.uuid}
* 字符串(32位MD5字符串)：${random.value}