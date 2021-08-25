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
#### 配置繁琐
虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。

- 一开始，Spring用XML配置，而且是很多XML配置。
- Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。
- Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

但是无论如何简化，需要编写的配置始终还是要写，所以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring实用，但它要求的回报也不少。

#### 依赖繁琐
项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发进度。

### 定义
Spring Boot 是一个javaweb的**开发框架**，Spring Boot **基于 Spring** 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。也就是说，它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。

### 核心
#### 自动配置 - 给你默认配置
Spring Boot 以约定大于配置的核心思想，**默认帮我们进行了很多设置**，多数 Spring Boot 应用只需要很少的 Spring 配置。

Spring Boot 的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。该过程是SpringBoot自动完成的。

简单来说，就是SpringBoot在运行时帮你把需要编写的一些配置都自动完成了——根据你开发的需要，提供了一整套“默认的配置”。

#### 起步依赖 - 帮你管理依赖
起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。

简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

#### 辅助功能 - 让你开箱即用
提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等。

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