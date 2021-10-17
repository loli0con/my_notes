# spring cloud

## 版本
[spring-cloud官网](https://spring.io/projects/spring-cloud)的**OVERVIEW**中查看spring-cloud和spring-boot的依赖关系，同时找到maven的配置。在**LEARN**中看到documentation，点击**Reference Doc.**，可以查看到&**Supported Boot Version**，<u>以此为最终依据</u>确定spring-boot的版本。

在[spring官网](https://start.spring.io/actuator/info)中能查看更详细的版本对应信息。

## 生态
Cloud：
* 注册中心
  * Eureka ❌
  * Zookeeper ✅
  * Consul ✅
  * Nacos ✅
* 服务调用-rest风格
  * Ribbo ✅
  * LoadBalancer ✅
* 服务调用-rpc风格
  * Feign ❌
  * OpenFeign ✅
* 服务降级
  * Hystrix ❌
  * resilience4j ⚠️
  * sentinel ✅
* 服务网关
  * Zuul ❌
  * Zuul2 ⚠️
  * gateway ✅
* 服务配置
  * Config ❌
  * Nacos ✅
* 服务总线
  * Bus ❌
  * Nacos ✅

## 工程构建
1. 创建project
2. 修改pom
3. 编写配置（YML）
4. 编写主启动类
5. 编写业务类