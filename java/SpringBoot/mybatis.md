# 整合mybatis

## 依赖
```xml
<dependency>
    <!-- 它依赖了jdbc启动器 -->
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

## 配置
### demo
```yml
mybatis:
  # 配置类型别名包扫描
  type-aliases-package: com.demo.pojo
  # sql语句映射文件路径，注意冒号后面没有空格，代表是一个整体
  mapper-locations:
    - classpath:mappers/*.xml
  # 字段返回值使用驼峰映射
  configuration:
    map-underscore-to-camel-case: true
```

## 设为Bean
有两种方式：
1. 在XxxMapper接口上添加@Mapper注解
2. 在启动类上添加数据访问接口包扫描注解@MapperScan，属性basePackages为*包路径*的数组。