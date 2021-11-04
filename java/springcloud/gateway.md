# Gateway
SpringCloud Gateway 是 Spring Cloud 的一个全新项目，基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式，且基于 Filter 链的方式提供了网关基本的功能，例如：反向代理，安全，监控/指标，熔断，和限流。

为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

## 核心概念
1. Route(路由)：路由是构建网关的基本模块，它由ID、目标URI、一系列的断言和过滤器组成，如果断言为true则匹配该路由。
2. Predicate(断言)：参考的是Java8的java.util.function.Predicate。开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由。
3. Filter(过滤)：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

![gateway+20211029212444](https://raw.githubusercontent.com/loli0con/picgo/master/images/gateway%2B20211029212444.png%2B2021-10-29-21-24-45)

## 工作流程
![gateway+20211029212611](https://raw.githubusercontent.com/loli0con/picgo/master/images/gateway%2B20211029212611.png%2B2021-10-29-21-26-12)

1. 客户端向 Spring Cloud Gateway 发出请求
2. Spring Cloud Gateway 在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。 
3. Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑：
* 在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
* 在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。 

## demo

### pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>             
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### yml
```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
      - id: payment_routh  # payment_route # 路由的 ID ，没有固定规则但要求唯一，建议配合服务名   
        uri: http://localhost:8001  # 匹配后提供服务的路由地址
        predicates:
        - Path=/payment/get/**  #  断言，路径相匹配的进行路由
      - id: payment_routh2  #payment_route # 路由的 ID ，没有固定规则但要求唯一，建议配合服务名
        uri: http://localhost:8001  # 匹配后提供服务的路由地址
        predicates:
        - Path=/payment/lb/** #  断言，路径相匹配的进行路由 
```

* routes是一个数组，表示该网关配置的多条路由规则
* id是这一条路有的标识符
* uri指定路由到的目标地址
* predicates是一个数组，表示该条路由的的断言规则

### 配置类
除了在配置文件yml中配置路由规则，还能通过在代码中注入RouteLocator的Bean（硬编码）的方式进行路由规则的配置。

```java
@Configuration
public class GateWayConfig
{  
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder){
        RouteLocatorBuilder.Builder routes = builder.routes();       
        routes.route("path_route_atguigu", r - > r.path("/guonei").uri("http://news.baidu.com/guonei")).build();  // 路由名称，predicate的path，目标uri
        return routes.build();
    }    
    @Bean   
    public  RouteLocator customRouteLocator2(RouteLocatorBuilder builder{
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("path_route_atguigu2", r - > r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
        return routes.build();
    }
}
```

### 动态路由
默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。uri的协议为lb，表示启用Gateway的负载均衡功能。

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
      - id: payment_routh  # payment_route # 路由的 ID ，没有固定规则但要求唯一，建议配合服务名   
        # uri: http://localhost:8001  # 匹配后提供服务的路由地址
        uri: lb://cloud-payment-service  # 匹配后提供服务的路由地址
        predicates:
        - Path=/payment/get/**  #  断言，路径相匹配的进行路由
      - id: payment_routh2  #payment_route # 路由的 ID ，没有固定规则但要求唯一，建议配合服务名
        # uri: http://localhost:8001  # 匹配后提供服务的路由地址
        uri: lb://cloud-payment-service  # 匹配后提供服务的路由地址
        predicates:
        - Path=/payment/lb/** #  断言，路径相匹配的进行路由 
```


## Predicate
Spring Cloud Gateway 包含许多内置的Route Predicate Factories，所有这些 Predicate 都匹配 HTTP 请求的不同属性。多种谓词工厂可以组合，并通过逻辑and。

Spring Cloud Gateway 创建 Route 对象时，使用 RoutePredicateFactory 创建 Predicate 对象，Predicate 对象可以赋值给 Route。

### Route Predicate Factories
中文文档：https://www.springcloud.cc/spring-cloud-greenwich.html#gateway-request-predicates-factories

在上述的中文文档里面有所有的Predicate和对应示例。  
其中ZonedDateTime的使用示例可以看如下代码：

```java
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class ZonedDateTimeDemo{
    public static void main(String[] args){
        ZonedDateTime zbj = ZonedDateTime.now();  // 默认时区
        System.out.println(zbj);
        // ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York"));  // 用指定时区获取当前时间
        // System.out.println(zny);
     }
}
```

## Filter
路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

Spring Cloud Gateway 内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生。

### 自定义Filter
```java
@Component //必须加，必须加，必须加
public class MyLogGateWayFilter implements GlobalFilter,Ordered{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
        System.out.println("time:" + new Date()+ "\t 执行了自定义的全局过滤器:" + "MyLogGateWayFilter" + "hello");
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if(uname == null ){ 
            System.out.println("**** 用户名为 null ，无法登录");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    
    @Override
    public int getOrder(){
        return 0;
    }
}
```