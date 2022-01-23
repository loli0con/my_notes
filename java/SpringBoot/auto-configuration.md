# 自动配置

## @SpringBootApplication
这个注解用来标注某一个类为SpringBoot的应用程序，截取@SpringBootApplication源码如下：
```java
@SpringBootConfiguration // 指定类是配置类
@EnableAutoConfiguration // 开启自动配置功能
@ComponentScan( // 组件扫描，扫描指定包路径，若路径下的类添加了Spring注解，则生成Bean
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
从源码可知：  
@SpringBootApplication ==等价于==  
@ComponentScan + @EnableAutoConfiguration + @SpringBootConfiguration

### @ComponentScan
作用：组件扫描，自动扫描并加载符合条件的组件或者bean，将这个bean定义加载到IOC容器中。

源码中的@ComponentScan没有指定包路径，则默认扫描当前类所在的包路径。因此我们编写的包/类/代码最好都放在启动类所在的目录下，否则不会被该注解扫描到。

#### excludeFilters属性
这个属性指定了一个*排除过滤器*数组，使用*排除过滤器*指明哪些类型的组件不参与到组件扫描中来。其中TypeExcludeFilter是提供给用户进行继承并自定义过滤规则的，而AutoConfigurationExcludeFilter是用来过滤掉会自动配置的配置类。

可以参考（我这里只说结论）：
1. https://segmentfault.com/a/1190000020550536
2. https://www.cnblogs.com/yql1986/p/9418419.html


### @SpringBootConfiguration
作用：SpringBoot的配置类，标注在某个类上，表示这是一个SpringBoot的配置类。

```java
@Configuration
public @interface SpringBootConfiguration {}
```

@SpringBootConfiguration ==等价于== @Configuration

这里的 @Configuration，说明这是一个Spring配置类——对应Spring的xml配置文件。

#### @Configuration
```java
@Component
public @interface Configuration {}
```
里面的 @Component 说明了启动类本身也是Spring中的一个组件，负责启动应用。

### @EnableAutoConfiguration
@EnableAutoConfiguration告诉SpringBoot开启自动配置功能，这样自动配置才能生效。
```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

#### @AutoConfigurationPackage
自动配置包
```java
// @Import({Registrar.class})
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```
其中：
* @import：Spring底层注解@import，给容器中导入一个组件。
* Registrar：定义默认的包扫描规则——将主启动类的所在包及包下面所有子包里面的所有组件扫描到Spring容器。

#### AutoConfigurationImportSelector
AutoConfigurationImportSelector：自动配置导入选择器。该类实现了ImportSelector接口，重写了selectImports方法，用于导入和自动配置相关的类。

![core+20210825105947](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210825105947.png%2B2021-08-25-10-59-50)

![core+20210825111234](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210825111234.png%2B2021-08-25-11-12-35)

![core+20210825120340](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210825120340.png%2B2021-08-25-12-03-42)

![core+20210825133931](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210825133931.png%2B2021-08-25-13-39-32)


## spring.factories
![kernel+20210822210935](https://raw.githubusercontent.com/loli0con/picgo/master/images/kernel%2B20210822210935.png%2B2021-08-22-21-09-38)

该文件里记录了SpringBoot中所有的自动配置类。
这些自动配置类，顾名思义，它们自动地帮我们完成了配置的步骤，简化了我们开发的工作。

### 举例
![core+20210823181014](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210823181014.png%2B2021-08-23-18-10-18)

@EnableConfigurationProperties：以一种“便利的方式”把带有@ConfigurationProperties注解的类注册到IOC容器当中。可以看到图中的HttpProperties类上有@ConfigurationProperties注解，这个HttpProperties类可以通过其他方式注册到容器中去，也可以通过这种“便利的方式”进行注册。

@ConfigurationProperties注解用于把properties等配置文件中的配置项-配置值绑定到类上。

![core+20210823183346](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210823183346.png%2B2021-08-23-18-33-49)

xxxxAutoConfigurartion是JavaConfig类（它有@Configuration注解），是SpringBoot提供的默认配置。


## 结论
1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，加载大量的自动配置类。
2. 将这些值作为自动配置类导入容器（通过反射实例化配置类），自动配置类生成，帮我们进行自动配置工作；
3. 整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；
4. 它会给容器中导入非常多的自动配置类（xxxAutoConfiguration），也就是给容器中导入这个场景需要的所有组件，并配置好这些组件；
5. 有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；


## 生效条件
自动配置类必须在一定的条件下才能生效。

### @Conditional派生注解
@Conditional派生注解 基于 Spring原生注解@Conditional 实现。

使用@Conditional派生注解来指定条件：只有当条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

常见的@Conditional派生注解，如下：
![core+20210823204720](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20210823204720.png%2B2021-08-23-20-47-21)


### 查看生效的配置
我们可以通过启用 debug=true 属性（在配置文件中），来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效。

在输出的日志中，会有如下结果：
* Positive matches:（自动配置类启用的：正匹配）
* Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）
* Unconditional classes:（没有条件的类）