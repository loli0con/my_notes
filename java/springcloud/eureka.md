# Eureka
eureka \[juˈriːkə]  
int. (因找到某物，尤指问题的答案而高兴)我发现了，我找到了

## 组成
Eureka包含两个组件：Eureka Server和Eureka Client，Eureka Server 提供服务注册服务，EurekaClient 通过注册中心进行访问。

## 调用
server通过http协议暴露接口，client通过RestTemplate访问接口。



## demo - 单eureka
### server
#### new module
#### 改pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency> 
```
#### 写yml
```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost    # eureka 服务端的实例名称
    client:
      register-with-eureka: false    # false 表示不向注册中心注册自己。
      fetch-registry: false    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      #service-url:    # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
        #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ 

```
#### 主启动
`@EnableEurekaServer`

### client
#### new module
#### 改pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency> 
```
#### 写yml
```yml
spring:
  application:
    name: cloud-service-name    # 微服务注册名配置说明

eureka:
  client:    # 表示是否将自己注册进 EurekaServer 默认为 true 。     
    register-with-eureka: true    # 是否从 EurekaServer 抓取已有的注册信息，默认为 true 。单节点无所谓，集群必须设置为 true 才能配合 ribbon 使用负载均衡     
    fetchRegistry: true    
    service-url:      
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka    #  集群版
      defaultZone: http://localhost:7001/eureka    #  单机版
```
#### 主启动
`@EnableEurekaClient`



## demo - eureka集群
server和client都要修改yml配置中的`defaultZone`为其他的eureka实例的地址，以逗号进行分割。

### client

#### 业务类
修改RestTemplate访问的URL地址，修改为“CLOUD-SERVICE-NAME”（服务提供者在eureka上注册的名字）。
#### 配置类
```java
@Configuration
public class ApplicationContextBean{
    @Bean
    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

## 微服务信息完善
完善前：
![eureka+20211020122014](https://raw.githubusercontent.com/loli0con/picgo/master/images/eureka%2B20211020122014.png%2B2021-10-20-12-20-15)

在yml配置文件中添加如下内容
```yml
eureka:
  instance:
    instance-id: custom_name    # 服务名修改：自定义的服务名
    prefer-ip-address: true      # ip信息提示：访问路径可以显示 IP 地址
```

完善后：
![eureka+20211020122130](https://raw.githubusercontent.com/loli0con/picgo/master/images/eureka%2B20211020122130.png%2B2021-10-20-12-21-31)


## 服务发现
对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息。

### 业务类
```java
@Resource
private DiscoveryClient discoveryClient;

public Object discovery(){
    List<String> services = discoveryClient.getServices();
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE"); // 获取某个微服务名称下面的微服务实例
    return this.discoveryClient;
} 
```

## 自我保护机制 - 高可用AP
默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。 

### 禁止自我保护
#### eureka server
```yml
eureka:
  server:
    # 关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```
#### eureka client
```yml
eureka:
  instance:
    #Eureka 客户端向服务端发送心跳的时间间隔，单位为秒 (默认是 30 秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka 服务端在收到最后一次心跳后等待时间上限，单位为秒 (默认是 90 秒) ，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```
