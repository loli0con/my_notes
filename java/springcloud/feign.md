# Feign
Feign是一个声明式的Web服务客户端，让编写Web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。

## 功能
Feign旨在使编写Java Http客户端变得更容易。 

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

## demo

### pom
```xml
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 主启动
```java
@EnableFeignClients
```

### 业务类

#### 服务提供者的接口（fegin client）
@FeignClient
```java
// 这是一个支付服务
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE") // value的值代表服务提供者在注册中心的微服务名称
public interface PaymentFeignService{
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
} 
```

#### 服务调用者的代码
```java
@RestControllerpublic
class OrderFeignController{
    @autowired
    private PaymentFeignService paymentFeignService;  // 使用被feign封装后的对象
    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id{         
        return paymentFeignService.getPaymentById(id);
    }
}
```

#### 服务提供者的代码
![feign+20211023135647](https://raw.githubusercontent.com/loli0con/picgo/master/images/feign%2B20211023135647.png%2B2021-10-23-13-56-49)



## 超时控制
默认Feign客户端只等待一秒钟，但是服务端处理需要超过1秒钟，导致Feign客户端不想等待了，直接返回报错。

为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制。

参考：https://www.cnblogs.com/kancy/p/13033021.html

### feign client configuration
```properties
# 默认开启
feign.httpclient.enabled=false
# 默认关闭
feign.okhttp.enabled=true
# 默认关闭
feign.hystrix.enabled=false
# 默认关闭
feign.sentinel.enabled=true

# default context 连接超时时间
feign.client.config.default.connectTimeout = 5000
# default context 读超时时间
feign.client.config.default.readTimeout = 10000

# 设置重试处理器，默认直接抛出异常
# feign.client.config.default.retryer = Class<Retryer>
# 设置日志级别，默认NONE
# feign.client.config.default.loggerLevel = FULL
```

### hystrix
```properties
# 全局设置超时：
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 30000
```

### ribbon configuration
```properties
# 连接超时时间，默认为1秒，该值会被FeignClient配置connectTimeout覆盖
ribbon.ConnectTimeout=5000
# 读超时时间，默认为1秒，该值会被FeignClient配置readTimeout覆盖
ribbon.ReadTimeout=5000
# 最大重试次数
ribbon.MaxAutoRetries=1
```

## 日志打印
Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。说白了就是对Feign接口的调用情况进行监控和输出。

### 日志级别
* NONE：默认的，不显示任何日志；
* BASIC：仅记录请求方法、URL、响应状态码及执行时间；
* HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
* FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。

### 配置日志
```java
@Configurationpublic
class FeignConfig{
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

### 配置feign客户端
```yml
logging:
  level:
    # feign 日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeignService: debug
```