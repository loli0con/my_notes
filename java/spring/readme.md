# Spring
Spring 是一款主流的 Java EE 轻量级开源框架，其目的是用于简化 Java 企业级应用的开发难度和开发周期。Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring 框架除了自己提供功能外，还提供整合其他技术和框架的能力。

在不同的语境中，Spring 所代表的含义是不同的，广义上的 Spring 泛指以 Spring Framework 为核心的 Spring 技术栈，狭义的 Spring 特指 Spring Framework，通常我们将它称为 Spring 框架。

## 介绍
* Spring是**分层**的Java SE/**EE应用**的**full-stack** **轻量级**开源**框架**。
* 它提供了一系列**底层容器**和**基础设施**
* 将开源世界中众多优秀的第三方框架和类库**整合**

## 核心
Spring 有两个核心部分:IOC 和 Aop
* **IoC**：Inverse of Control 的简写，译为“控制反转”，指把创建对象过程交给 Spring 进行管理。
* **AOP**：Aspect Oriented Programming 的简写，译为“面向切面编程”。AOP 用来封装多个类的公共行为，将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，减少系统的重复代码，降低模块间的耦合度。另外，AOP 还解决一些系统层面上的问题，比如日志、事务、权限等。

IOC的核心是工厂模式，AOP的核心是代理模式。

### IOC容器
容器是一种为某种特定组件的运行提供必要支持的一个软件环境。通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。

Spring的核心就是提供了一个IoC容器，它可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。

## 特点
Spring的特点： 
* 非侵入式：使用 Spring Framework 开发应用程序时，Spring 对应用程序本身的结构影响非常小。对领域模型可以做到零污染；对功能性组件也只需要使用几个简单的注解进行标记，完全不会破坏原有结构，反而能将组件结构进一步简化。这就使得基于 Spring Framework 开发应用程序时结构清晰、简洁优雅。
* 控制反转：IoC——Inversion of Control，翻转资源获取方向。把自己创建资源、向环境索取资源变成环境将资源准备好，我们享受资源注入。
* 面向切面编程：AOP——Aspect Oriented Programming，在不修改源代码的基础上增强代码功能。
* 容器：Spring IoC 是一个容器，因为它包含并且管理组件对象的生命周期。组件享受到了容器化的管理，替程序员屏蔽了组件创建过程中的大量细节，极大的降低了使用门槛，大幅度提高了开发效率。
* 组件化：Spring 实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用 XML 和 Java 注解组合这些对象。这使得我们可以基于一个个功能明确、边界清晰的组件有条不紊的搭建超大型复杂应用系统。
* 一站式：在 IoC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库。而且 Spring 旗下的项目已经覆盖了广泛领域，很多方面的功能性需求可以在 Spring Framework 的基础上全部使用 Spring 来实现。

## 衍生
在Spring Framework基础上，又诞生了Spring Boot、Spring Cloud、Spring Data、Spring Security等一系列基于Spring Framework的项目。

## 体系
Spring框架采用的是分层架构，它一系列功能要素被分成20个模块：

![readme+20230926094127](https://raw.githubusercontent.com/loli0con/picgo/master/images/readme%2B20230926094127.png%2B2023-09-26-09-41-28)

### Core Container核心容器
[IOC](ioc.md)容器：
* Beans模块：提供了BeanFactory，工厂模式的实现，Spring将所有管理的对象称为Bean。
* Core模块：提供了Spring框架的基本组成部分，包括IoC和DI依赖注入的功能。
* Context模块：访问和配置Bean对象的上下文对象，核心容器。如：ApplicationContext
* Expression模块：spring表达式语言

### AOP
* [AOP](aop.md)模块：提供了面向切面编程的功能，整合ASM，CGLib，JDK Proxy
* Aspects模块：提供了与AspectJ框架集成功能
* Instrument模块：动态Class Loading模块

### Data Access/Integration数据访问与集成
* [JDBC](jdbc.md)模块：提供了JDBC的支持，如：JdbcTemplate
* ORM模块：对主流的ORM框架提供了支持，如：JPA，JDO，Hibernate等
* Transactions模块：事务管理，支持声明式事务
* OXM模块：对象与xml文件的映射框架
* JMS模块：Spring对Java Message Service(java消息服务)的封装，用于服务之间相互通信

### Web
* spring-web：最基础的web支持，建立于spring-context之上，通过servlet或listener来初始化IOC容器
* spring-webmvc：实现web mvc
* spring-websocket：与前端的全双工通信协议
* spring-webflux：Spring 5.0提供的，用于取代传统java servlet，非阻塞式Reactive Web框架，异步，非阻塞，事件驱动的服务

### 其他模块
* Test模块：提供了对单元测试和集成测试的支持，主要是对junit的封装
* Spring-messaging：spring 4.0提供的，为Spring集成一些基础的报文传送服务


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
    <!-- redis数据库驱动包 -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
    <!--mysql数据库驱动包-->
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
    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    <!-- 引入mybatis的 pagehelper 分页插件 -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.1.2</version>
    </dependency>
    <!-- spring-mybatis支持包 -->
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
        <version>1.7.30</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>



    <!-- ================  spring   整合  Redis================== -->
    <!-- 1、java连接Redis的jar包，也就是使用jedis -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.3</version>
    </dependency>
    <!-- 2、spring整合Redis的jar包，注意版本的对应关系 -->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>2.1.6.RELEASE</version>
    </dependency>
    <!-- JSON工具：jackson-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!--jackson xml的转换工具 -->
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
        <version>2.9.9</version>
    </dependency>



    <!-- commons -->
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
    <!--字符串处理工具类包-->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.5</version>
    </dependency>
    <!-- commons-beanutils -->
    <dependency>
        <groupId>commons-beanutils</groupId>
        <artifactId>commons-beanutils</artifactId>
        <version>1.9.3</version>
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

<!-- 配置Maven静态资源导出（非必要） -->
<build>
   <resources>
        <resource>
            <!-- 把src/main/java目录下的配置也导出 -->
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

log4j.category.org.springframework=info

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
jdbc.url=jdbc:mysql://localhost:3306/day666?useSSL=false&useUnicode=true&characterEncoding=utf8
# jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.driver=com.mysql.jdbc.Driver
jdbc.initialSize=3
jdbc.maxActive=10
```

### web.xml
```xml

<!--配置web.xml加载applicationContext.xml创建spring框架的IOC容器-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
    <!-- classpath*: 除了在当前项目的classpath中查找，也会从依赖项目中的classpath中查找 -->
    <!-- <param-value>classpath*:applicationContext.xml</param-value> -->
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




<!-- 整合redis -->
<!--1. 创建redis的独立配置类，配置主机与端口-->
<bean id="standaloneConfiguration" class="org.springframework.data.redis.connection.RedisStandaloneConfiguration">
    <!--redis的主机ip-->
    <property name="hostName" value="localhost"/>
    <!--redis使用的端口-->
    <property name="port" value="6379"/>
</bean>
<!--2. 创建Jedis连接工厂-->
<bean id="factory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg ref="standaloneConfiguration"/>
</bean>
<!--3创建redisTemplate模板对象，使用该对象可以操作redis-->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <!--1.配置连接工厂-->
    <property name="connectionFactory" ref="factory"/>
    <!--2. 配置键序列化器-->
    <property name="keySerializer">
        <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
    </property>

    <!--2. 配置值序列化器-->
    <property name="valueSerializer">
        <bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer"/>
    </property>
</bean>



</beans>
```