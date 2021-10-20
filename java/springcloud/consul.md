# Consul
Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。

它具有很多优点:
* 基于 raft 协议，比较简洁；
* 支持健康检查
* 同时支持 HTTP 和 DNS 协议
* 支持跨数据中心的 WAN 集群
* 提供图形界面跨平台
* 支持 Linux、Mac、Windows 

它的功能:
* 服务治理
* 配置中心
* 控制总线
* 健康监测
* KV存储
* 多数据中心
* 可视化Web界面

## demo
通过以下地址可以访问Consul的首页：http://localhost:8500
### pom
```xml
<!--SpringCloud consul-server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

### yml
```yml
###consul 服务端口号
server:
  port: 8006
spring:
  application:
    name: consul-provider-payment
  ####consul 注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

### 主启动
```java
@EnableDiscoveryClient
```

