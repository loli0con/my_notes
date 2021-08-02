# Spring

## 文档
* [官方文档](ioc.mdhttps://docs.spring.io/spring-framework/docs/5.2.16.RELEASE/spring-framework-reference/core.html)
* [中文文档(非官方)](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference)

## 下载地址
https://repo.spring.io/release/org/springframework/spring/

## 介绍
* Spring是**分层**的Java SE/**EE应用**的**full-stack** **轻量级**开源**框架**。
* 它提供了一系列**底层容器**和**基础设施**
* 将开源世界中众多优秀的第三方框架和类库**整合**

### 容器
容器是一种为某种特定组件的运行提供必要支持的一个软件环境。通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。

Spring的核心就是提供了一个IoC容器，它可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。

## 核心
Spring 有两个核心部分:IOC 和 Aop
* IOC:控制反转，把创建对象过程交给 Spring 进行管理
* Aop:面向切面，不修改源代码进行功能增强

IOC的核心是工厂模式，AOP的核心是代理模式。

## 特点
Spring的特点： 
* 方便**解耦**，**简化**开发
* **Aop** 编程支持
* 方便程序**测试**
* 方便和其他框架进行**整合**
* 方便进行**事务**操作
* 降低 API 开发难度

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
<!-- 使用Jdk8 -->
<properties> 
    <maven.compiler.source>8</maven.compiler.source>  
    <maven.compiler.target>8</maven.compiler.target>
    <!--将依赖的jar包的版本号抽取到properties全局属性标签内，目的是可以重用版本号-->
    <spring_version>5.2.0.RELEASE</spring_version>
</properties>

<!-- 常用依赖 -->
<dependencies>
    <!-- Spring -->
    <!--Spring基础包，统一版本号使用5.2.0-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring_version}</version>
    </dependency>
    <!--SpringMVC-->
    <!-- 会间接依赖spring-context, spring-web等 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring_version}</version>
    </dependency>

    <!-- 数据库访问 -->
    <!--spring的jdbc支持包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring_version}/version>
    </dependency>
    <!--数据库驱动包-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
        <!-- <version>5.1.47</version> -->
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
        <version>1.1.16</version>
        <!-- <version>1.1.12</version> -->
    </dependency>
    <!--声明式事务支持-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring_version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.2</version>
        <!-- <version>1.3.2</version> -->
    </dependency>

    <!-- AOP支持 -->
    <!-- AspectJ切面表达式支持 -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.13</version>
        <!-- <version>1.8.7</version> -->
        <scope>runtime</scope>
    </dependency>

    <!-- 单元测试 -->
    <!--junit4-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <!--junit5-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.5.2</version>
        <scope>test</scope>
    </dependency>
    <!-- spring测试包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring_version}</version>
        <scope>test</scope>
    </dependency>
    

    <!-- 日志 -->
    <!-- Log4j 依赖 -->
    <dependency>
        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.6.4</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>


    <!-- JSON工具 -->
    <!-- Jackson -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.8</version>
    </dependency>


    <!--文件上传-->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.3</version>
    </dependency>
    <!--文件下载操作流使用-->
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>


    <!--servlet-api，文件操作、Jackson都依赖它-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <!-- <version>4.0.1</version> -->
        <scope>provided</scope>
    </dependency>
    <!--jsp-->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <!-- 
            This artifact was moved to:
            javax.servlet.jsp » javax.servlet.jsp-api
        -->
        <!-- <artifactId>javax.servlet.jsp-api</artifactId> -->
        <version>2.2</version>
        <scope>provided</scope>
    </dependency>
    <!--jstl-->
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
        <scope>provided</scope>
    </dependency>


</dependencies>

<!-- 配置Maven静态资源导出（非必要，只是用于预防静态资源导出失败） -->
<build>
   <resources>
       <resource>
           <directory>src/main/java</directory>
           <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
           </includes>
           <filtering>false</filtering>
       </resource>
       <resource>
           <directory>src/main/resources</directory>
           <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
           </includes>
           <filtering>false</filtering>
       </resource>
   </resources>
</build>
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
# mysql8+需要额外配置时区参数 
# serverTimezone=Asia/Shanghai
# serverTimezone=utc
jdbc.url=jdbc:mysql://localhost:3306/day666?useSSL=true&useUnicode=true&characterEncoding=utf8
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.initialSize=3
jdbc.maxActive=10
```

### web.xml
```xml

<!--配置web.xml加载applicationContext.xml创建spring框架的IOC容器-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--配置ContextLoaderListener会在服务器最早启动的时候读取全局参数contextConfigLocation的配置创建spring框架IOC容器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>


<!-- 前端控制器 -->
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--设置加载springmvc.xml配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--启动时创建Servlet-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<!--配置乱码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--设置码表-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Spring 配置文件
适用于applicationContext.xml/springMVC.xml
```xml
<!-- 常用的命名空间 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/c
       http://www.springframework.org/schema/c/spring-c.xsd
       http://www.springframework.org/schema/p
       http://www.springframework.org/schema/p/spring-p.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd
       ">



<!-- 容易遗漏的 -->

<!-- 开启属性注解支持 -->
<context:annotation-config/>
<!-- 包下注解扫描 -->
<context:component-scan base-package="com.demo"/>
<!-- 读取配置文件(示例中为druid配置文件)，使用文件中的属性值 -->
<context:property-placeholder location="classpath:druid.properties"/>
<!-- 开启生成代理对象 -->
<aop:aspectj-autoproxy>
<!--配置注解式事务的驱动，指定事务管理器 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
<!-- 让Spring MVC不处理静态资源 -->
<mvc:default-servlet-handler />
<!-- mvc注解驱动 -->
<mvc:annotation-driven />
<!-- 配置视图解析器:前缀与后缀 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/pages/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>



<!--
spring整合mybatis的步骤
    1.加载外部jdbc.properties配置文件
    2.创建Druid连接池对象
    3.配置spring整合mybatis的SqlSessionFactoryBean对象：目的接管mybatis创建的对象加入IOC容器
    4.配置mybatis映射扫描的包
    5.配置spring的声明式事务
-->

<!--加载外部jdbc.properties配置文件
    注意：location配置的路径中必须有“classpath” 否则报错
-->
<context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
<!-- <context:property-placeholder location="classpath:druid.properties"></context:property-placeholder> -->

<!--创建Druid连接池对象-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${jdbc.driver}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>

<!--
配置spring整合mybatis的SqlSessionFactoryBean对象：目的接管mybatis创建的对象加入IOC容器
-->
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--注入连接池对象dataSource-->
    <property name="dataSource" ref="dataSource"></property>
    <!--接管：包别名设置
    <property name="typeAliasesPackage" value="com.demo.entity"></property>-->
    <!--加载mybatis-config.xml里面配置-->
    <property name="configLocation" value="classpath:mybatis-config.xml"></property>
</bean>

<!--配置mybatis映射扫描的包: spring会利用sqlSession到指定包下创建Dao的代理对象加入IOC容器-->
<bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 注入sqlSessionFactory -->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean">
    <property name="basePackage" value="com.demo.dao"/>
</bean>

<!--配置spring的声明式事务-->
<!--a.创建事务管理器,注意：注入连接池-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!--b.配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="query*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="search*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<!--c.aop配置：将通知给到切入点去增强实现声明式事务-->
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com..service.impl.*.*(..))"></aop:advisor>
</aop:config>

```