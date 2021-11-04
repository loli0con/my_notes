# Config

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。 

## 架构
SpringCloud Config分为服务端和客户端两部分。 
  
服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。


## 功能
1. 集中管理配置文件
2. 根据运行环境进行动态配置
3. 运行期间动态调整配置
4. 配置信息以REST接口的形式暴露
5. 与GitHub整合


## demo - 服务端
### pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### yml
```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center  # 注册进 注册中心 的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@github.com:zzyybs/springcloud-config.git  #GitHub 上面的 git 仓库名字
            #### 搜索目录
            search-paths:
              - springcloud-config
            #### 读取分支
      label:  master
```

### 主启动
```java
@EnableConfigServer
```

### 测试
![config+20211030142032](https://raw.githubusercontent.com/loli0con/picgo/master/images/config%2B20211030142032.png%2B2021-10-30-14-20-33)

根据读取规则，访问服务端（rest接口）：
1. /{label}/{application}-{profile}.yml
2. /{application}-{profile}.yml
3. /{application}/{profile}\[/{label}]

推荐使用第一种，而且在编写配置文件时，也使用第一种

## demo - 客户端

### pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<!-- 动态刷新需要用到的依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### yml
```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config 客户端配置
    config:
      label: master  # 分支名称
        name: config  # 配置文件名称
        profile: dev  # 读取后缀名称   上述 3 个综合： master 分支上 config-dev.yml 的配置文件被读取 http://config-3344.com:3344/master/config-dev.yml
        uri: http://localhost:3344  # 配置中心地址 k
```

### 动态刷新
避免每次更新配置都要重启客户端微服务

#### yml
```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 业务类
```java
@RefreshScope
```

#### 测试 - 刷新配置
使用POST方法向监控端点发送请求，如下：  
curl -X POST "http://localhost:3355/actuator/refresh"

此时配置会被动态刷新。
