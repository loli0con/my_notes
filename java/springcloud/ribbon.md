# Ribbon
Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。主要功能是提供客户端的软件负载均衡算法和服务调用。

Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

## LB
* 集中式LB：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5；也可以是软件，如nginx),由该设施负责把访问请求通过某种策略转发至服务的提供方
* 进城内LB：将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

## 步骤
Ribbon在工作时分成两步： 
1. 第一步先选择EurekaServer，它优先选择在同一个区域内负载较少的server。
2. 第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。

其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。 


## demo
### pom
```xml
<!-- 需要注意，可能别的组建会自动引入ribbon，例如eureka。这时可能会发生冲突 -->
<dependency>
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId> 
</dependency> 
```

### 内置的策略
![ribbon+20211022013806](https://raw.githubusercontent.com/loli0con/picgo/master/images/ribbon%2B20211022013806.png%2B2021-10-22-01-38-07)

* RoundRobinRule：轮询
* RandomRule：随机
* RetryRule：先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
* WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
* BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
* AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例
* ZoneAvoidanceRule：默认规则，复合判断server所在区域的性能和server的可用性选择服务器

### 业务类
如[eureka笔记](eureka.md)中的业务类，使用RestTemplate访问其他微服务。

### 配置类

#### 启用Ribbon
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

#### 替换策略
这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，即不能处于主启动类的同级或下级。

```java
@Configuration
public class MySelfRule{
    @Bean
    public IRule myRule(){
        return new RandomRule(); // 定义为随机
    }
}  
```

#### 主启动类
```java
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", // 微服务命名
configuration=MySelfRule.class)  // 导入策略
```