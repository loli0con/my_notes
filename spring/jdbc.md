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

