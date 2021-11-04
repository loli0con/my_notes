# Nacos
Nacos: Dynamic Naming and Configuration Service

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

![nacos+20211030145630](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030145630.png%2B2021-10-30-14-56-31)

## 安装
1. 下载地址：https://github.com/alibaba/nacos/releases
2. 解压安装包，直接运行bin目录下的startup.cmd -m standalone
3. 命令运行成功后直接访问http://localhost:8848/nacos，默认账号密码都是nacos

## demo - 注册中心
把微服务注册上去
### pom
```xml
<!--SpringCloud ailibaba nacos -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency> 
```

tips，nacos集成了ribbon：
![nacos+20211030153110](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030153110.png%2B2021-10-30-15-31-10)

### yml
```yml
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848  # 配置 Nacos 地址
```

### 主启动
```java
@EnableDiscoveryClient
```

## 配置中心对比
![nacos+20211030154304](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030154304.png%2B2021-10-30-15-43-05)

### 模式选择
如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如 Spring cloud 和 Dubbo 服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。 

如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须，K8S服务和DNS服务则适用于CP模式。
CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

切换CP：
`curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP' `

## demo - 配置中心
获取配置中心的配置
### pom
```xml
<!--nacos-config-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency> 
```

### yml
application.yml
```yml
spring:
  profiles:
    active: dev # 表示开发环境 
```

bootstrap.yml
```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848  #Nacos 服务注册中心地址
    config:
      server-addr: localhost:8848  #Nacos 作为配置中心地址
      file-extension: yaml  #指定yaml 格式的配置 

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} 
```

### 业务类
在控制器类加入 @RefreshScope 注解使当前类下的配置支持 Nacos 的动态刷新功能。

客户端自带动态刷新，不需要再像[Config](config.md#测试%20-%20刷新配置)一样刷新配置。

### 测试
添加配置：Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则。

![nacos+20211030162735](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030162735.png%2B2021-10-30-16-27-36)

![nacos+20211030162835](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030162835.png%2B2021-10-30-16-28-37)


## demo - 配置管理
Nacos提供Group和Namespace来解决“多环境多项目”的问题。

![nacos+20211030165835](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030165835.png%2B2021-10-30-16-58-38)

* Nacos默认的命名空间是public，Namespace主要用来实现隔离。
* Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去。
* Service就是微服务；一个Service可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。
* 最后是Instance，就是微服务的实例。


### yml
```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848  #Nacos 服务注册中心地址
    config:
      server-addr: localhost:8848  #Nacos 作为配置中心地址
      file-extension: yaml  #指定yaml 格式的配置 
      group: TEST_GROUP
      namespace:  5da1dccc-ee26-49e0-b8e5-7d9559b95ab0  #命名空间ID
```


## 集群 + 持久化
![nacos+20211030170339](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030170339.png%2B2021-10-30-17-03-40)

默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL的存储。

### 过程
1. 执行nacos/conf/nacos-mysql.sql脚本
2. 修改nacos/conf/application.properties配置
   ```properties
   spring.datasource.platform=mysql

   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=123456
   ```
3. 修改nacos/conf/cluster.conf
   ![nacos+20211030171052](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211030171052.png%2B2021-10-30-17-10-53)
4. 修改/nacos/bin/startup.sh，让它支持“-P 端口号”启动
   ![nacos+20211101091209](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211101091209.png%2B2021-11-01-09-12-11)
5. 修改nginx配置
   ![nacos+20211101091614](https://raw.githubusercontent.com/loli0con/picgo/master/images/nacos%2B20211101091614.png%2B2021-11-01-09-16-16)