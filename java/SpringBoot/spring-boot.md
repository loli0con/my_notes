# Spring Boot

![spring-boot+20220112175232](https://raw.githubusercontent.com/loli0con/picgo/master/images/spring-boot%2B20220112175232.png%2B2022-01-12-17-52-36)

概括来说：以 简单的方式 构建 强大的应用程序。

## Spring Initializr - 创建项目
使用Spring Initializr创建一个Spring boot项目（也可以自己以Maven的方式创建项目），可以勾选一个最基本的Spring Web依赖（也可以不这么做）。

![core+20220112170727](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20220112170727.png%2B2022-01-12-17-07-30)

## spring-boot-dependencies - 管理依赖
*Spring boot项目/当前项目*的pom（这里是example1），其父母为spring-boot-starter-parent。

![core+20220112172912](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20220112172912.png%2B2022-01-12-17-29-14)
```xml
<!-- Spring boot项目/当前项目.pom -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version> <!-- 版本号不同也不影响 -->
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```
关于[<relativePath/>](https://blog.csdn.net/gzt19881123/article/details/105255138)的作用。


查看*spring-boot-starter-parent项目*的pom，其父母为spring-boot-dependencies。
![core+20220112173414](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20220112173414.png%2B2022-01-12-17-34-16)
```xml
<!-- spring-boot-starter-parent-2.6.1.pom -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.1</version> <!-- 版本号不同也不影响 -->
</parent>
```

**spring-boot-dependencies**才是真正管理SpringBoot应用里面所有依赖版本的地方，它是SpringBoot的版本控制中心。
![core+20220112173822](https://raw.githubusercontent.com/loli0con/picgo/master/images/core%2B20220112173822.png%2B2022-01-12-17-38-26)

从这里我们可以知道：  
SpringBoot为我们提供了一套最优的依赖包版本搭配组合，并且使用了传递依赖功能。我们想使用哪个功能（比如Redis），只需要在pom文件加入一个启动器依赖的坐标（不需要指定版本号），它会自动帮我们把相关的所有依赖包都传递依赖进来，且不会存在版本冲突问题。

## starter - 导入依赖
SpringBoot将所有的功能场景都抽取出来，做成一个个的starter（启动器）。在项目中引入这些starter，所有相关的依赖都会导入进来，因此要用什么功能就导入什么样的场景启动器即可。我们也可以自己自定义starter。

**项目的需求 *==对应到==>*  要实现的功能  *==抽象为==>*  场景  *==封装为==>*  启动器**

### 常见的启动器
```xml
<!--web启动器，将项目变成web项目，集成了springMVC、tomcat、json等相关功能支持-->
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

还有：*消息队列*、*AOP织入*、*数据源*、*JDBC*、*redis*、*缓存*、*模版引擎*、*日志*、*JSON*等项目中需要用到的、涉及到的东西，都被做成了启动器供程序员使用。

## SpringApplication.run - 启动程序
```java
// @SpringBootApplication 来标注一个主程序类
// 说明这是一个Spring Boot应用，即是由我们编写的SpringBoot程序。
@SpringBootApplication
public class Example1Application {
   public static void main(String[] args) {
     //将 SpringBoot 应用启动：通过反射加载了我们所编写SpringBoot程序
      SpringApplication.run(Example1Application.class, args);
   }
}
```


