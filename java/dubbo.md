# dubbo
Apache Dubbo 是一款高性能、轻量级的开源服务框架。

Apache Dubbo |ˈdʌbəʊ| 提供了六大核心能力：
* 面向接口代理的高性能RPC调用
* 智能容错和负载均衡
* 服务自动注册和发现
* 高度可扩展能力
* 运行期流量调度
* 可视化的服务治理与运维

## 架构
![dubbo+20210901174841](https://raw.githubusercontent.com/loli0con/picgo/master/images/dubbo%2B20210901174841.png%2B2021-09-01-17-48-42)

### 节点角色说明
| 节点      | 角色说明                               |
| --------- | -------------------------------------- |
| Provider  | 暴露服务的服务提供方                   |
| Consumer  | 调用远程服务的服务消费方               |
| Registry  | 服务注册与发现的注册中心               |
| Monitor   | 统计服务的调用次数和调用时间的监控中心 |
| Container | 服务运行容器                           |

### 调用关系说明
0. 服务容器负责启动，加载，运行服务提供者。
1. 服务提供者在启动时，向注册中心注册自己提供的服务。
2. 服务消费者在启动时，向注册中心订阅自己所需的服务。
3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 环境搭建
### 注册中心 zookeeper
1. 下载zookeeper
2. 解压zookeeper
3. 基于zoo_sample.cfg创建并修改zoo.cfg配置文件
   1. 设置数据存储的目录：dataDir
   2. 设置zookeeper的端口号：clientPort

### 管理控制台 dubbo-admin
1. 下载dubbo-admin
2. 修改dubbo-admin配置
   1. 修改src\main\resources\application.properties指定zookeeper地址
3. 打包：mvn clean package -Dmaven.test.skip=true 
4. 运行：java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
5. 访问：默认端口7001，账号:root，密码:root

### 简单监控中心 dubbo-monitor-simple
1. 下载dubbo-monitor-simple
2. 修改dubbo-monitor-simple\src\main\resources\conf\dubbo.properties配置
   1. 修改dubbo.registry.address=zookeeper://127.0.0.1:2181
   2. 修改dubbo.jetty.port=8080
3. 打包dubbo-monitor-simple：mvn clean package -Dmaven.test.skip=true
4. 解压 tar.gz 文件，并运行start.bat
5. 启动访问8080

####  监控中心配置
```xml
<!-- 所有服务配置连接监控中心，进行监控统计 -->
<!-- 监控中心协议，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心 -->
<dubbo:monitor protocol="registry"></dubbo:monitor>
```

## 依赖
```xml
<!-- 引入dubbo -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.6.2</version>
</dependency>
<!-- 由于我们使用zookeeper作为注册中心，所以需要操作zookeeper
dubbo 2.6及以后的版本引入curator操作zookeeper
-->
<!-- zookeeper的api管理依赖，封装了一些高级特性 -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>

<!-- 提供了高级的API来简化ZooKeeper的使用 -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>

<!-- zookeeper依赖 -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
</dependency>
```

## demo
https://dubbo.apache.org/zh/docsv2.7/user/quick-start/
### 服务提供者

#### 定义服务接口
```java
package org.apache.dubbo.demo;

public interface DemoService {
    String sayHello(String name);
}
```

#### 在服务提供方实现接口
```java
package org.apache.dubbo.demo;

public interface DemoService {
    String sayHello(String name);
}
```

#### 用 Spring 配置声明暴露服务
```xml
<!--指定当前服务/应用的名字  -->
<dubbo:application name="hello-world-app" />
<!--指定注册中心的地址  -->
<dubbo:registry address="zookeeper://127.0.0.1:2181" />
<!--指定通信的规则：用dubbo协议在20880端口暴露服务  -->
<dubbo:protocol name="dubbo" port="20880" />
<!-- 声明需要暴露的服务 -->
 <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
<!-- 上面的ref指向下面的id -->
<!-- 服务的实现类，和本地bean一样实现服务 -->
<bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
```

#### 加载 Spring 配置
```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Provider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml"});
        context.start();
        System.in.read(); // 按任意键退出
    }
}
```
#### 查看
在dubbo-admin的“服务治理-提供者-服务名”里面查看。

### 服务消费者

#### 通过 Spring 配置引用远程服务
```xml
<!-- 应用名 -->
<dubbo:application name="consumer-of-helloworld-app" />
<!-- 指定注册中心地址 -->
<dubbo:registry address="zookeeper://127.0.0.1:2181" />
<!-- 生成远程服务代理，可以和本地bean一样使用demoService-->
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
```

#### 加载Spring配置，并调用远程服务
```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.apache.dubbo.demo.DemoService;
 
public class Consumer {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("consumer.xml");
        context.start();
        DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法
        System.out.println( hello ); // 显示调用结果
    }
}
```


## 用法示例
### 配置优先级
-D > XML > Properties
![dubbo+20210901211233](https://raw.githubusercontent.com/loli0con/picgo/master/images/dubbo%2B20210901211233.png%2B2021-09-01-21-12-34)

### 启动时检查
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/preflight-check/)
Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 check="true"。

### zookeeper宕机与dubbo直连
注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯。

同时消费者可以通过直连提供者，[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/explicit-target/)。

### 重试次数
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/%E9%87%8D%E8%AF%95%E6%AC%A1%E6%95%B0%E9%85%8D%E7%BD%AE/)
失败自动切换，当出现失败，重试其它服务器，但重试会带来更长延迟。

#### 注解配置
```java
@Service(timeout = 30000, retries = 0)
public class UserApiImpl implements UserApi {
    // dubbo对外提供的接口实现类
}
```

### 超时打断
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/provider-timeout-release/)
由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间。


#### xml配置
由于官网提供了注解形式的配置参考，这里补充一下xml格式的配置。
##### 消费者
```xml
<!-- 全局超时配置 -->
<dubbo:consumer timeout="5000" />

<!-- 指定接口以及特定方法超时配置 -->
<dubbo:reference interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:reference>
```

##### 服务端
```xml
<!-- 全局超时配置 -->
<dubbo:provider timeout="5000" />

<!-- 指定接口以及特定方法超时配置 -->
<dubbo:provider interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:provider>
```

### 多版本
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/multi-versions/)

灰度发布：当出现新功能时，会让一部分用户先使用新功能，用户反馈没问题时，再将所有用户迁移到新功能。

dubbo 中使用 version 属性来设置和调用同一个接口的不同版本

#### 注解配置
```java
// 服务端实现类的注解
@Service(version = "1.0") // version指定服务版本
@Service(version = "2.0") // version指定服务版本

// 消费端代理接口的注解
@Reference(version = "1.0")  
@Reference(version = "2.0")  // 修改版本后重启
```

### 负载均衡
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/loadbalance/)
在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用。

#### 注解实现
```java
// 服务端实现类的注解
@Service(weight = 100)
@Service(weight = 200)

// 消费端代理接口的注解
@Reference(loadbalance = "random")
```

### 集群容错
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/fault-tolerent-strategy/)

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

#### 注解实现
```java
@Reference(cluster = "failover")
```


### 服务降级
[官方文档](https://dubbo.apache.org/zh/docsv2.7/user/examples/service-downgrade/)

当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。

#### 注解配置
```java
// 表示不发起远程调用，直接返回null
@Reference(mock = "force:return null")
// 服务的方法调用在失败后，再返回 null 值，不抛异常
@Reference(mock = "fail:return null")
```