# 整合dubbo

## demo

### 服务提供者

#### 依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--指定SpringBoot继承的父项目-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <groupId>cn.itcast</groupId>
    <artifactId>dubbo-service-user</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
         <!--web启动器，传递依赖了tomcat-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--dubbo的起步依赖-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.5</version>
        </dependency>
        
        <!-- zookeeper的api管理依赖 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
        
        <!-- zookeeper依赖 -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>

    </dependencies>
</project>
```

#### 配置
```yml
# springboot项目对外暴漏的端口（web项目就是访问端口）
server:
  port: 9001

dubbo:
  application:
    # 项目名称，用于计算服务的依赖关系。（区分不同项目的服务）
    name: dubbo-service-user
  # 指定zk注册中心地址
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    # 对外暴漏的服务使用的协议：dubbo协议, 官网推荐
    name: dubbo
    # 对外暴漏服务的端口
    port: 20881
  # dubbo包扫描，会扫描dubbo提供的@Service注解
  scan:
    base-packages: cn.itcast.api
```

#### 编写Api接口
```java
package cn.itcast.api;
/**
 * dubbo对外提供的服务的接口
 */
public interface UserApi {
    String sayHi(String name);
}
```

#### 编写Api接口实现类
```java
package cn.itcast.api.impl;
import cn.itcast.api.UserApi;
import org.apache.dubbo.config.annotation.Service;  
/**
 * 使用dubbo提供的@Service注解
 * 1. 导入的包：org.apache.dubbo.config.annotation.Service
 * 2. 注意：不要导入spring的包 org.springframework.stereotype.Service
 */
@Service
public class UserApiImpl implements UserApi {
    @Override
    public String sayHi(String name) {
        return "hi," + name;
    }
}
```

#### 启动类
```java
package cn.itcast;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

### 服务消费者

#### 依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <groupId>cn.itcast</groupId>
    <artifactId>order-web-manager</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--dubbo的起步依赖-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.5</version>
        </dependency>
        
        <!-- zookeeper的api管理依赖 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
        
        <!-- zookeeper依赖 -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>
    </dependencies>
</project>
```

#### 配置
```yml
server:
  port: 8080
dubbo:
  # 项目名称，用于计算服务的依赖关系。（区分不同项目的服务）
  application:
    name: order-web-manager
  # 注册中心地址
  registry:
    address: zookeeper://127.0.0.1:2181
```

#### 编写api接口
```java
package cn.itcast.api;
/**
 * 服务消费者：
 * 1、这里的接口一定要与服务提供者一样
 * 2、接口包名称、接口名称、接口方法名称、方法参数个数类型、返回值都要一样。
 */
public interface UserApi {
    String sayHi(String name);
}
```


#### 控制器
```java
package cn.itcast.controller;

import cn.itcast.api.UserApi;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("order")
public class OrderController {

    /**
     * @Reference
     * 1、导入dubbo提供的注解：org.apache.dubbo.config.annotation.Reference
     * 2、用于对指定的接口生成代理对象，通过代理封装远程调用相关代码，实现远程调用
     */
    @Reference
    private UserApi userApi;

    @GetMapping
    public String hello(){
        // 执行RPC远程调用, 获取dubbo服务返回结果. 在浏览器显示
        String result = userApi.sayHi("Tom");
        return result;
    }
}
```

#### 启动类
```java
package cn.itcast;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```