# Spring + JDBC

## JdbcTemplate
JdbcTemplate是Spring提供的一个模板类，它是对jdbc的封装。用于数据库持久层的操作，它的特点是：简单、方便。

它简化了JDBC的使用，并有助于避免常见错误。它执行核心的JDBC工作流程，我们只需要写SQL语句，并且从中获取结果就可以了。

### 常用方法
|JdbcTemplate类|说明|
|---|---|
|public JdbcTemplate(DataSource dataSource)|作用：创建模板对象<br />参数：数据源，从中获取连接对象|
|public int update(String sql, Object…args)|作用：实现增删改的操作<br />参数1：SQL语句<br />参数2：替换占位符的值<br />返回值：返回影响的行数|
|\<T> T queryForObject(String sql, <br />RowMapper\<T> rowMapper, Object... args)|作用：查询1条记录<br />参数1：SQL语句<br />参数2：指定映射关系的接口(RowMapper)<br />参数3：替换占位符的值<br />返回值：查询的那条记录的对象|
|List\<T> query(String sql, RowMapper\<T> rowMapper, Object... args)|查询多条记录|

### RowMapper接口
#### 接口方法
```java
T mapRow(ResultSet rs, int rowNum) throws SQLException
```
* 作用：将结果集封装成一个对象
* 参数：
  1. 询出来的结果集
  2. 这是第几行
* 返回值：封装好的对象
#### 实现类
```java
public  BeanPropertyRowMapper(Class\<T> mappedClass)
```
* 作用：实现了上面的接口
* 参数：要封装的实体类类型

### 示例
```java
DriverManagerDataSource ds = new DriverManagerDataSource();
ds.setUsername("root");
ds.setPassword("root");
ds.setUrl("jdbc:mysql:///day666");
ds.setDriverClassName("com.mysql.jdbc.Driver");

//2.创建JdbcTemplate对象，传入数据源
JdbcTemplate jdbcTemplate = new JdbcTemplate(ds);

//3.调用JdbcTemplate的方法访问数据库，查询一条记录。
//参数1：SQL语句，参数2：映射对象，参数3：要替换的一个或多个占位符
//RowMapper是一个接口，作用：将结果集封装成一个实体类对象。我们使用它提供的实现类
//BeanPropertyRowMapper是RowMap接口的实现类，前提：表的列名与实体类的属性名相同（下划线自动转驼峰），参数就是要封装的实体类类型
Student student = jdbcTemplate.queryForObject("select * from student where id=?", new BeanPropertyRowMapper<>(Student.class), 1);
System.out.println(student);
```

## DataSource

### DriverManagerDataSource
Spring提供的数据源：org.springframework.jdbc.datasource.DriverManagerDataSource
#### 使用
```xml
<bean class="org.springframework.jdbc.datasource.DriverManagerDataSource" id="dataSource">
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    <property name="url" value="jdbc:mysql:///day36"/>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
</bean>
```

### c3p0
com.mchange.v2.c3p0.ComboPooledDataSource
#### 使用
```xml
<!-- 使用c3p0的连接池 -->
<bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
    <!-- 有三个属性的名字不同 -->
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="jdbcUrl" value="jdbc:mysql:///day666"/>
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
</bean>
```

### druid
com.alibaba.druid.pool.DruidDataSource
#### 使用
```xml
<bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    <property name="url" value="jdbc:mysql:///day666"/>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
</bean>
```

## 事务
事务主要用在业务层。业务层中一个方法会多次调用DAO层中增删改，所有的方法必须全部执行成功，如果有一个方法执行失败，就要进行回滚。要么全部成功，要么全部失败。

Spring 的事务控制都是基于 AOP 的，它既可以使用编程的方式实现，也可以使用配置的方式实现。

Spring 框架为我们提供了一组事务控制的接口，这组接口是在spring-tx-版本.RELEASE.jar中。

![jdbc+20210731152036](https://i.loli.net/2021/07/31/vaoGNbjYMlUp4we.png)

### 编程式事务
通过编写代码实现的事务管理，包括定义事务的开始，正常执行后事务提交和异常时的事务回滚。

### 声明式事务
开发者无须通过编程的方式来管理事务，只需在配置文件中进行相关的事务规则声明，就可以将事务规则应用到业务逻辑中。

#### API
##### PlatformTransactionManager
事务的顶层接口，提供了操作事务的常用方法，方法由Spring容器去调用：
* TransactionStatus getTransaction(TransactionDefinition definition)：获取事务状态信息
* void commit(TransactionStatus status)：提交事务
* void rollback(TransactionStatus status)：回滚事务

通常使用org.springframework.jdbc.datasource.DataSourceTransactionManager实现类。

##### TransactionDefinition
定义了事务的隔离级别、传播行为、超时时间等。

###### 事务的隔离级别
事务指定一个隔离级别，该隔离级别定义一个事务必须与由其他事务进行的资源或数据更改相隔离的程度。
![jdbc+image-20200228094252193](https://i.loli.net/2021/07/31/dJIhVi6NOl14mCt.png)

###### 事务的传播行为
事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行。

![jdbc+20210731174120](https://i.loli.net/2021/07/31/WQ9MbyjJa4iLRCY.png)

###### TransactionStatus
提供了获取事务状态的方法，事务是否是新的，事务是否完成，事务是否是只读等。

#### xml实现
```xml
<!-- 配置事务管理器 -->
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
    <!--注入数据源(需要先配置好数据源)-->
    <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 声明式事务切面的配置，指定事务管理器 -->
<tx:advice id="interceptor" transaction-manager="transactionManager">
    <tx:attributes>
        <!--
        name：指定哪些方法需要使用事务（可以使用通配符*），
        以及使用事务的规则：
            propagation：事务传播行为（默认为REQUIRED）
            isolation：事务的隔离级别
            read-only：指定事务是否只读（默认为false）
            timeout：超时的时间（默认为false）
            rollback-for：指定触发事务回滚的异常类（如果指定多个类用逗号分隔）
            no-rollback-for：指定不触发事务回滚的异常类
        -->
        <tx:method name="transfer" read-only="false" propagation="REQUIRED" isolation="DEFAULT"/>
        <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
    </tx:attributes>
</tx:advice>

<!-- AOP的配置 -->
<aop:config>
    <!-- 切面表达式的配置 -->
    <aop:pointcut id="pt" expression="execution(* com.demo.service..*.*(..))"/> <!-- com.demo.service下的所有方法 -->
    <!-- 配置上面的事务 -->
    <aop:advisor advice-ref="interceptor" pointcut-ref="pt"/>
</aop:config>
```

#### 注解实现
##### xml配置
```xml
<!-- 配置事务管理器 -->
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
    <!--注入数据源-->
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--配置注解式事务的驱动，指定事务管理器 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

##### @Transactional
放在类、接口或方法上，让这个方法使用事务：
* 类上面：表示这个类中所有的方法都使用事务
* 方法上面：表示这个方法使用事务
* 接口：表示只要实现这个接口的所有子类中所有方法都使用事务
* 接口中方法：表示实现这个接口的子类中这个方法使用事务

| 参数名称      | 描述                                             |
| ------------- | ------------------------------------------------ |
| propagation   | 传播行为                                         |
| isolation     | 隔离级别                                         |
| readOnly      | 是否只读                                         |
| timeout       | 超时时间，默认是-1，表示不超时                   |
| rollbackFor   | 哪些异常会进行回滚，默认只对非运行时异常进行回滚 |
| noRollbackFor | 哪些异常不进行回滚                               |

