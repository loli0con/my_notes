# 整合JDBC
对于数据访问层，无论是 SQL(关系型数据库) 还是 NOSQL(非关系型数据库)，Spring Boot 底层都是采用 Spring Data 的方式进行统一处理。

## 依赖
项目的pom文件需要引入如下内容：
```xml
<!-- jdbc启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<!-- 数据库驱动包 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 配置
数据源的所有自动配置都在：DataSourceAutoConfiguration类中。
```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
    # type: 指定数据源
```

### 测试类
```java
@SpringBootTest
class SpringbootDataJdbcApplicationTests {

    //DI注入数据源
    @Autowired
    DataSource dataSource;

    @Test
    public void contextLoads() throws SQLException {
        //看一下默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //关闭连接
        connection.close();
    }
}
```


## JDBCTemplate
Spring 本身对原生的JDBC做了轻量级的封装，即JdbcTemplate。数据库操作的所有 CRUD 方法都在 JdbcTemplate 中。JdbcTemplate 的自动配置在 JdbcTemplateAutoConfiguration 类中。

Spring Boot 不仅提供了默认的数据源（Hikari），同时默认已经配置好了 JdbcTemplate 放在了容器中，程序员只需自己注入即可使用。

### API
JdbcTemplate主要提供以下几类方法：
* execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句；
* update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；
* query方法及queryForXXX方法：用于执行查询相关语句；
* call方法：用于执行存储过程、函数相关语句。