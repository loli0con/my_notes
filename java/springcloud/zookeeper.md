# Zookeeper
zookeeper是一个分布式协调工具，可以实现注册中心功能。zookeeper服务器取代Eureka服务器（因此请先阅读[eureka笔记](./eureka.md)），zk作为服务注册中心。

## demo
服务节点是临时节点，zk是强一致（CP）。
### pom
```xml
<!-- SpringBoot 整合 zookeeper 客户端  -->
<dependency>
    <groupId> org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <!-- 如果发生版本冲突，则剔除包中的zookeeper。 -->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--上面剔除了版本不匹配的zookeeper后，这里要重新添加zookeeper，示例的版本为3.4.9，请根据实际环境进行替换。 -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency> 
```

可能会出现版本冲突，请确认`spring-cloud-starter-zookeeper-discovery`包中的zookeeper包的版本，和实际运行的zookeeper软件是否为同一个版本号。

### yml
```yml
#表示注册到 zookeeper 服务器的服务提供者端口号
server:
  port: 8004

# 服务别名 ---- 注册 zookeeper 到注册中心名称
spring:
  application:
    name: cloud-provider-name
  cloud:
    zookeeper:
      connect-string: 127.0.0.1:2181
```

### 主启动
```java
@EnableDiscoveryClient
// 该注解用于向使用 consul 或者 zookeeper 作为注册中心时注册服务
```

## 备注
消费者的配置和eureka笔记本中的一模一样，仅仅是替换了注册中心而已。

如果需要支持zookeeper注册中心集群，在client程序的配置的`spring.cloud.zookeeper.connect-string`里，配置多个ip+port，以逗号分隔即可。