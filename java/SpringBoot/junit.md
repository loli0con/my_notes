# 整合junit

## 依赖

```xml
<!-- 配置test启动器(自动整合spring-test、junit) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 示例
```java
// 指定运行环境是Spring环境
@RunWith(SpringRunner.class)
// 如果测试类所在目录是启动类所在目录的同级子级下可以省略启动类
// @SpringBootTest(classes = {MyApplication.class})
@SpringBootTest
public class UserServiceTest { /*省略代码*/}
```