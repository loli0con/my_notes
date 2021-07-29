# Spring

## 文档
* https://docs.spring.io/spring-framework/docs/5.2.16.RELEASE/spring-framework-reference/core.html

## 介绍
* Spring是**分层**的Java SE/**EE应用**的**full-stack** **轻量级**开源**框架**。
* 它提供了一系列底层容器和**基础设施**
* 将开源世界中众多优秀的第三方框架和类库**整合**

### 容器
容器是一种为某种特定组件的运行提供必要支持的一个软件环境。通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。

Spring的核心就是提供了一个IoC容器，它可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。

## 衍生
在Spring Framework基础上，又诞生了Spring Boot、Spring Cloud、Spring Data、Spring Security等一系列基于Spring Framework的项目。

## 体系
Spring框架采用的是分层架构，它一系列功能要素被分成20个模块：
<!-- TODO -->
![readme+20210728120928](https://i.loli.net/2021/07/28/eporQKfDtzMI8Gy.png)

### Core Container核心容器
* Beans模块：提供了BeanFactory，工厂模式的实现，Spring将所有管理的对象称为Bean。
* Core模块：提供了Spring框架的基本组成部分，包括IoC和DI依赖注入的功能。
* Context模块：访问和配置Bean对象的上下文对象，核心容器。如：ApplicationContext

### Data Access/Integration数据访问与集成
* JDBC模块：提供了JDBC的支持，如：JdbcTemplate
* ORM模块：对主流的ORM框架提供了支持，如：JPA，JDO，Hibernate等
* Transaction模块：支持声明式事务的管理

### Web
* Servlet模块：包含了SpringMVC和REST Web Services实现的Web应用程序
* Web模块：提供了基本的Web开发集成特性，如：文件上传，收发邮件等。

### 其他模块
* AOP模块：提供了面向切面编程的功能
* Aspects模块：提供了与AspectJ框架集成功能
* Test模块：提供了对单元测试和集成测试的支持

## 实用模版

### pom.xml
```xml
<dependencies>
        <!--Spring支持包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        

        <!-- 数据库访问 -->
        <!--spring的jdbc支持包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        <!--数据库驱动包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
            <!-- <version>5.1.22</version> -->
        </dependency>
        <!--连接池-->
        <!-- C3P0数据源 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.4</version>
        </dependency>
        <!-- Druid数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
            <!-- <version>1.1.12</version> -->
        </dependency>
        

        <!-- 单元测试 -->
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!-- spring测试包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.0.RELEASE</version>
            <scope>test</scope>
        </dependency>
        

        <!-- 日志 -->
        <!-- Log4j 依赖 -->
        <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.7</version>
        </dependency>

    </dependencies>
```

### log4j.properties
```properties
log4j.rootLogger=debug, stdout

log4j.category.org.springframework=debug

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
```

### druid.properties
```properties
jdbc.username=root
jdbc.password=root
jdbc.url=jdbc:mysql://localhost:3306/day666?characterEncoding=utf8
jdbc.driverClassName=com.mysql.jdbc.Driver
```