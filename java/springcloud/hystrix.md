# Hystrix

## 分布式系统面临的问题 
复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

### 服务雪崩
多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”。

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

## 问题的解决者
Hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

### 断路器
“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

## 重要概念

### 服务降级 - 系统有限的资源的合理协调
服务降级一般是指在服务器压力剧增的时候，根据实际业务使用情况以及流量，对一些服务和页面有策略的不处理或者用一种简单的方式进行处理，从而释放服务器资源的资源以保证核心业务的正常高效运行。

以下情况会导致服务降级：
  1. 程序运行异常
  2. 超时
  3. 服务熔断触发服务降级
  4. 线程池/信号量打满也会导致服务降级

### 服务熔断 - 应对雪崩效应的链路自我保护机制
服务熔断是服务降级的一种特殊情况，是防止服务雪崩而采取的措施。  
当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。

### 服务限流

## demo
### pom
```xml
<!--hystrix-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### demo - 服务降级
服务降级可以作用在服务端和客户端，推荐在客户端配置服务降级。
#### 服务端（提供者）

##### 业务类
```java
@HystrixCommand(  // 降级配置注解
    fallbackMethod="paymentInfo_TimeOutHandler",  // 失败时调用的方法（兜底方案）
    commandProperties={
        @HystrixProperty(
            name="execution.isolation.thread.timeoutInMilliseconds",  // 设定超时时长
            value="3000"
        )
    }
)
public String paymentInfo_TimeOut(Integer id){
    int second=5;
    try {TimeUnit.SECONDS.sleep(second);}
    catch(InterruptedException e){e.printStackTrace();}
    return"线程池:"+Thread.currentThread().getName()+"paymentInfo_TimeOut,id:"+id+"\t"+"O(∩_∩)O耗费秒:"+second;
}

public String paymentInfo_TimeOutHandler(Integer id){  // 失败时调用的方法
    return "/(ㄒoㄒ)/调用支付接口超时或异常：\t"+"\t当前线程池名字"+Thread.currentThread()getName();
}
```

##### 启动类
```java
@EnableCircuitBreaker
```

#### 客户端（消费者）

##### 配置
```yml
feign:
  hystrix:
    enabled: true  # 在 Feign 中开启 Hystrix
```

##### 业务类
```java
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
@HystrixCommand(
    fallbackMethod="paymentTimeOutFallbackMethod",
    commandProperties={
        @HystrixProperty(
            name="execution.isolation.thread.timeoutInMilliseconds",
            value="1500"
          )
    }
)
public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
    String result=paymentHystrixService.paymentInfo_TimeOut(id);
    return result;
}

public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
    return"我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
} 
```

##### 启动类
```java
@EnableHystrix
```

#### 默认的兜底方法
![hystrix+20211026165927](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211026165927.png%2B2021-10-26-16-59-29)

#### 业务的解耦方案
上述的兜底方法和业务方法（处理请求的controller）放在一起，容易导致混乱

##### feign接口
```java
@Component
@FeignClient(value="CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback=PaymentFallbackService.class)  // 指定fallback类
public interface PaymentFeignClientService{
    @GetMapping ("/payment/hystrix/{id}")
    public String getPaymentInfo(@PathVariable("id") Integer id);
}
```

##### fallback类
```java
@Component //必须加
public class PaymentFallbackService implements PaymentFeignClientServic{
    @Override
    public String getPaymentInfo(Integer id){
        return "服务调用失败，提示来自：cloud-consumer-feign-order80";
    }
}
```

### demo - 服务熔断
![hystrix+20211026173417](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211026173417.png%2B2021-10-26-17-34-18)


#### 提供者/消费者
```java
@HystrixCommand(fallbackMethod="paymentCircuitBreaker_fallback",
    commandProperties={
        @HystrixProperty(name="circuitBreaker.enabled",value="true"), // 开启断路器    
        @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value="10"), // 请求总数阀值
        @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",value="10000"), // 快照时间窗
        @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value="60"), // 错误百分比阀值
    }
    // 在 10000 毫秒内，如果收到超过 10 个请求，且其中有超过 60% 的请求触发了服务降级，则进入熔断状态
)
public String paymentCircuitBreaker @PathVariable("id") Integer id){
    if(id<0){
        throw new RuntimeException("******id 不能负数");
    }
    String serialNumber=IdUtil.simpleUUID();
    return Thread.currentThread().getName()+"\t"+"调用成功，流水号:"+serialNumber;
}

public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id){
    return"id 不能负数，请稍后再试，/(ㄒoㄒ)/~~ id:"+id;
} 
```

#### 所有配置
```java
commandProperties={
    // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
    @HystrixProperty(name="execution.isolation.strategy",value="THREAD"),

    // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
    @HystrixProperty(name="execution.isolation.semaphore.maxConcurrentRequests",value="10"),

    // 配置命令执行的超时时间
    @HystrixProperty(name="execution.isolation.thread.timeoutinMilliseconds",value="10"),

    // 是否启用超时时间
    @HystrixProperty(name="execution.timeout.enabled",value="true"),

    // 执行超时的时候是否中断
    @HystrixProperty(name="execution.isolation.thread.interruptOnTimeout",value="true"),

    // 执行被取消的时候是否中断
    @HystrixProperty(name="execution.isolation.thread.interruptOnCancel",value="true"),

    // 允许回调方法执行的最大并发数
    @HystrixProperty(name="fallback.isolation.semaphore.maxConcurrentRequests",value="10"),

    // 服务降级是否启用，是否执行回调函数
    @HystrixProperty(name="fallback.enabled",value="true"),

    // 是否启用断路器
    @HystrixProperty(name="circuitBreaker.enabled",value="true"),

    // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
    // 如果滚动时间窗（默认10秒）内仅收到了19个请求，即使这19个请求都失败了，断路器也不会打开。
    @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value="20"),

    // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
    // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,
    // 就把断路器设置为"打开"状态，否则就设置为"关闭"状态。
    @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value="50"),

    // 该属性用来设置当断路器打开之后的休眠时间窗。休眠时间窗结束之后，
    // 会将断路器置为"半开"状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为"打开"状态，
    // 如果成功就设置为"关闭"状态。
    @HystrixProperty(name="circuitBreaker.sleepWindowinMilliseconds",value="5000"),

    // 断路器强制打开
    @HystrixProperty(name="circuitBreaker.forceOpen",value="false"),

    // 断路器强制关闭
    @HystrixProperty(name="circuitBreaker.forceClosed",value="false"),

    // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
    @HystrixProperty(name="metrics.rollingStats.timeinMilliseconds",value="10000"),

    // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
    // 设置的时间窗长度拆分成多个"桶"来累计各度量值，每个"桶"记录了一段时间内的采集指标。
    // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
    @HystrixProperty(name="metrics.rollingStats.numBuckets",value="10"),

    // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false,那么所有的概要统计都将返回 -1。
    @HystrixProperty(name="metrics.rollingPercentile.enabled",value="false"),

    // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
    @HystrixProperty(name="metrics.rollingPercentile.timeInMilliseconds",value="60000"),

    // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
    @HystrixProperty(name="metrics.rollingPercentile.numBuckets",value="60000"),
    // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
    // 就从最初的位置开始重写。例如，将该值设置为100,滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
    // 那么该 “桶” 中只保留最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
    @HystrixProperty(name="metrics.rollingPercentile.bucketSize",value="100"),
    // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、错误百分比）的间隔等待时间。
    @HystrixProperty(name="metrics.healthSnapshot.intervalinMilliseconds",value="500"),
    // 是否开启请求缓存
    @HystrixProperty(name="requestCache.enabled",value="true"),
    //HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
    @HystrixProperty(name="requestLog.enabled",value="true"),
},
    
threadPoolProperties={
    // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
    @HystrixProperty(name="coreSize",value="10"),

    // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，// 否则将使用 LinkedBlockingQueue 实现的队列。
    @HystrixProperty(name="maxQueueSize",value="-1"),

    // 该参数用来为队列设置拒绝阈值。通过该参数，即使队列没有达到最大值也能拒绝请求。
    // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue 
    // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
    @HystrixProperty(name="queueSizeRejectionThreshold",value="5"),
}
```


## 工作流程
![hystrix+20211029204456](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211029204456.png%2B2021-10-29-20-44-57)


1. 创建 HystrixCommand（用在依赖的服务返回单个操作结果的时候）或 HystrixObserableCommand（用在依赖的服务返回多个操作结果的时候）对象。
2. 命令执行。其中 HystrixComand 实现了下面前两种执行方式；而 HystrixObservableCommand 实现了后两种执行方式：
   * execute()：同步执行，从依赖的服务返回一个单一的结果对象，或是在发生错误的时候抛出异常。
   * queue()：异步执行，直接返回一个Future对象，其中包含了服务执行结束时要返回的单一结果对象。
   * observe()：返回 Observable 对象，它代表了操作的多个结果，它是一个 Hot Obserable（不论"事件源"是否有"订阅者"，都会在创建后对事件进行发布，所以对于 Hot Observable 的每一个"订阅者"都有可能是从"事件源"的中途开始的，并可能只是看到了整个操作的局部过程）。
   * toObservable()：同样会返回 Observable 对象，也代表了操作的多个结果，但它返回的是一个Cold Observable（没有"订阅者"的时候并不会发布事件，而是进行等待，直到有"订阅者"之后才发布事件，所以对于 Cold Observable 的订阅者，它可以保证从一开始看到整个操作的全部过程）。
3. 若当前命令的请求缓存功能是被启用的，并且该命令缓存命中，那么缓存的结果会立即以 Observable 对象的形式返回。
4. 检查断路器是否为打开状态。如果断路器是打开的，那么Hystrix不会执行命令，而是转接到 fallback 处理逻辑（第8步）；如果断路器是关闭的，检查是否有可用资源来执行命令（第5步）。
5. 线程池/请求队列/信号量是否占满。如果命令依赖服务的专有线程池和请求队列，或者信号量（不使用线程池的时候）已经被占满，那么 Hystrix 也不会执行命令，而是转接到 fallback 处理逻辑（第8步）。
6. Hystrix 会根据我们编写的方法来决定采取什么样的方式去请求依赖服务。
   * HystrixCommand.run()：返回一个单一的结果，或者抛出异常。
   * HystrixObservableCommand.construct()：返回一个Observable 对象来发射多个结果，或通过 onError 发送错误通知。
7. Hystrix会将 "成功"、"失败"、"拒绝"、"超时" 等信息报告给断路器，而断路器会维护一组计数器来统计这些数据。断路器会使用这些统计数据来决定是否要将断路器打开，来对某个依赖服务的请求进行"熔断/短路"。
8. 当命令执行失败的时候，Hystrix 会进入 fallback 尝试回退处理，我们通常也称该操作为"服务降级"。而能够引起服务降级处理的情况有下面几种：
   * 第4步：当前命令处于"熔断/短路"状态，断路器是打开的时候。
   * 第5步：当前命令的线程池、请求队列或者信号量被占满的时候。
   * 第6步：HystrixObservableCommand.construct() 或 HystrixCommand.run() 抛出异常的时候。
9. 当Hystrix命令执行成功之后，它会将处理结果直接返回或是以 Observable 的形式返回。


tips：如果我们没有为命令实现降级逻辑或者在降级处理逻辑中抛出了异常，Hystrix 依然会返回一个 Observable 对象，但是它不会发射任何结果数据，而是通过 onError 方法通知命令立即中断请求，并通过onError()方法将引起命令失败的异常发送给调用者。


## Dashboard
除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。

### pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

<!-- dashboard服务和被监控的服务需要加入这一项依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>         
</dependency>
```

### yml
```yml
server:
    port: 9001  # 监控页面的端口，呆会访问  localhost:9001  这个url能看到dashboard。
```

### 主启动
```java
@EnableHystrixDashboard
```

### 监控端点
这里本不用配置，如果会遇到错误：“Unable to connect to Command Metric Stream.”，则需要进行如下配置。

新版本Hystrix需要在主启动类（被监控的服务）中指定监控路径：
```java
/** 
    此配置是为了服务监控而配置，与服务容错本身无关，springcloud 升级后的坑。

    ServletRegistrationBean 因为 springboot 的默认路径不是 "/hystrix.stream"，只要在自己的项目里配置上下面的 servlet 就可以了
*/

@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new  HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean =  new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
} 
```

### 监控页面
![hystrix+20211029211220](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211029211220.png%2B2021-10-29-21-12-22)

#### 内容解析
![hystrix+20211029211531](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211029211531.png%2B2021-10-29-21-15-32)

![hystrix+20211029211701](https://raw.githubusercontent.com/loli0con/picgo/master/images/hystrix%2B20211029211701.png%2B2021-10-29-21-17-03)