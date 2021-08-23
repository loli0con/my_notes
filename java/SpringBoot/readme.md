# Spring Boot

## Spring
Spring是为了解决企业级应用开发的复杂性而创建的，简化开发。

### 简化方式
为了降低Java开发的复杂性，Spring采用了以下4种关键策略：
1. 基于POJO的轻量级和最小侵入性编程，所有**东西都是bean**；
2. 通过**IOC**，依赖注入（DI）和面向接口实现松耦合；
3. 基于切面（**AOP**）和惯例进行声明式编程；
4. 通过切面和**模版**减少样式代码，RedisTemplate，xxxTemplate；

## SpringBoot
### 背景
随着 Spring 不断的发展，涉及的领域越来越多，项目整合开发需要配合各种各样的文件，慢慢变得不那么易用简单，违背了最初的理念，甚至人称**配置地狱**。Spring Boot 正是在这样的一个背景下被抽象出来的开发框架，目的为了让大家更容易的使用 Spring 、更容易的集成各种常用的中间件、开源软件。

### 定义
Spring Boot 是一个javaweb的**开发框架**，Spring Boot **基于 Spring** 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。也就是说，它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。

### 核心
Spring Boot 以约定大于配置的核心思想，**默认帮我们进行了很多设置**，多数 Spring Boot 应用只需要很少的 Spring 配置。同时它集成了大量常用的第三方库配置（例如 Redis、MongoDB、Jpa、RabbitMQ、Quartz 等等），Spring Boot 应用中这些第三方库几乎可以零配置的开箱即用。

### 优点
Spring Boot的主要优点：
* 为所有Spring开发者更**快**的入门
* **开箱即用**，提供各种**默认配置**来简化项目配置
* 内嵌式容器简化Web项目（例如内嵌Tomcat）
* 没有冗余代码生成和XML配置的要求

## 备忘录📕

### pom.xml
```xml
<!-- 父依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <!-- web场景启动器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- springboot单元测试 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <!-- 剔除依赖 -->
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- 打包插件 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!--跳过项目运行测试用例-->
                <skipTests>true</skipTests>
            </configuration>
        </plugin>
    </plugins>
</build>

```