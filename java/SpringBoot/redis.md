# 整合redis

## 依赖
```xml
<!-- 配置redis启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 配置
```yml
# 配置Redis
spring:
  redis:
    host: localhost # 主机
    port: 6379      # 端口
```

