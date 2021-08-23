# 核心
关于SpringBoot最重要的一点：“通过**预设的配置**来解决开发过程中的**复杂性**问题，实现快速启动、开箱即用”。

要掌握SpringBoot关键在于掌握SpringBoot里面的预设配置，  
要学习SpringBoot重点就是学习配置的艺术！

## 配置的艺术

### 开箱即用的pom
当前项目的pom
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

spring-boot-starter-parent项目的pom
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

**spring-boot-dependencies**才是真正管理SpringBoot应用里面所有依赖版本的地方，它是SpringBoot的版本控制中心。它提供了许多的默认构建（默认版本）以供实际项目使用。


#### 启动器starter
SpringBoot将所有的功能场景都抽取出来，做成一个个的starter（启动器）。在项目中引入这些starter，所有相关的依赖都会导入进来，因此要用什么功能就导入什么样的场景启动器即可。我们未来也可以自己自定义starter。

**项目的需求 *==对应到==>*  要实现的功能  *==抽象为==>*  场景  *==封装为==>*  启动器**

##### 常见的功能场景/启动器
```xml
<!-- springboot自身 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<!-- web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 单元测试 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

还有：消息队列、AOP织入、数据源、JDBC、redis、缓存、模版引擎、日志、JSON等项目中需要用到、需要处理、需要实现的东西，都被做成了启动器供我们使用。


### 超级简单的启动类
```java
//@SpringBootApplication 来标注一个主程序类
//说明这是一个Spring Boot应用，即是由我们编写的SpringBoot程序。
@SpringBootApplication
public class SpringbootApplication {
   public static void main(String[] args) {
     //将 SpringBoot 应用启动：通过反射加载了我们所编写SpringBoot程序
      SpringApplication.run(SpringbootApplication.class, args);
   }
}
```
这是一个简单的启动类，但它并不简单，它的复杂性暗藏于@SpringBootApplication中。

#### @SpringBootApplication
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {
        @Filter(
            type = FilterType.CUSTOM,
            classes = {TypeExcludeFilter.class}
        ), 
        @Filter(
            type = FilterType.CUSTOM,
            classes = {AutoConfigurationExcludeFilter.class}
        )
    }
)
public @interface SpringBootApplication {
    // ......
}
```

SpringBootApplication = ComponentScan + EnableAutoConfiguration + SpringBootConfiguration

#### @ComponentScan
作用：组件扫描，自动扫描并加载符合条件的组件或者bean，将这个bean定义加载到IOC容器中。

#### @SpringBootConfiguration
作用：SpringBoot的配置类，标注在某个类上，表示这是一个SpringBoot的配置类。

```java
@Configuration
public @interface SpringBootConfiguration {}
```

SpringBootConfiguration = Configuration

这里的 @Configuration，说明这是一个Spring配置类——对应Spring的xml配置文件。

##### @Configuration
```java
@Component
public @interface Configuration {}
```
里面的 @Component 这就说明，启动类本身也是Spring中的一个组件而已，负责启动应用。

#### @EnableAutoConfiguration
@EnableAutoConfiguration告诉SpringBoot开启自动配置功能，这样自动配置才能生效。
```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

##### @AutoConfigurationPackage
自动配置包
```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```
@import：Spring底层注解@import，给容器中导入一个组件。
Registrar.class作用：将主启动类的所在包及包下面所有子包里面的所有组件扫描到Spring容器。

##### AutoConfigurationImportSelector
AutoConfigurationImportSelector：自动配置导入选择器，这个类中有一个这样的方法：
```java

// 获得候选的配置
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    //这里的getSpringFactoriesLoaderFactoryClass（）方法
    //返回的就是我们最开始看的启动自动导入配置文件的注解类；EnableAutoConfiguration
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```
这个方法又调用了SpringFactoriesLoader类的静态方法loadFactoryNames()：
```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    //这里它又调用了 loadSpringFactories 方法
    return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    //获得classLoader ， 我们返回可以看到这里得到的就是EnableAutoConfiguration标注的类本身
    MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        try {
            //去获取一个资源 "META-INF/spring.factories"
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            LinkedMultiValueMap result = new LinkedMultiValueMap();

            //将读取到的资源遍历，封装成为一个Properties
            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                Iterator var6 = properties.entrySet().iterator();

                while(var6.hasNext()) {
                    Entry<?, ?> entry = (Entry)var6.next();
                    String factoryClassName = ((String)entry.getKey()).trim();
                    String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                    int var10 = var9.length;

                    for(int var11 = 0; var11 < var10; ++var11) {
                        String factoryName = var9[var11];
                        result.add(factoryClassName, factoryName.trim());
                    }
                }
            }

            cache.put(classLoader, result);
            return result;
        } catch (IOException var13) {
            throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
        }
    }
}
```
可以看到，获取了一个spring.factories资源。

#### spring.factories
![kernel+20210822210935](https://raw.githubusercontent.com/loli0con/picgo/master/images/kernel%2B20210822210935.png%2B2021-08-22-21-09-38)

该文件里记录了SpringBoot中所有的自动配置类。
这些自动配置类，顾名思义，它们自动地帮我们完成了配置的步骤，简化了我们开发的工作。

##### 举例
![kernel+20210822205908](https://raw.githubusercontent.com/loli0con/picgo/master/images/kernel%2B20210822205908.png%2B2021-08-22-20-59-11)
在SpringBoot中有一个*WebMvcAutoConfiguration配置类*（负责整个WebMVC的配置），  
WebMvcAutoConfiguration配置类里面有个*WebMvcAutoConfigurationAdapter配置类*（负债总额部分WebMVC的配置），  
WebMvcAutoConfigurationAdapter配置类上的注解中的*WebMvcProperties类*描述了配置的内容（配置项以及配置值）。

这些都是JavaConfig配置类（它们都有@Configuration注解），都是SpringBoot提供的默认配置。

#### 结论
1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值。
2. 将这些值作为自动配置类导入容器（通过反射示例化配置类），自动配置类就生效，帮我们进行自动配置工作；
3. 整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；
4. 它会给容器中导入非常多的自动配置类（xxxAutoConfiguration），就是给容器中导入这个场景需要的所有组件，并配置好这些组件；
5. 有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；


### 自动
