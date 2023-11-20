# 基于AOP的声明式事务控制

## 事务分类
Spring的事务分为: 编程式事务控制 和 声明式事务控制。

|事务控制方式|解释|
|---|---|
|编程式事务控制|Spring提供了事务控制的类和方法，使用编码的方式对业务代码进行事务控制，事务控制代码和业务操作代码耦合到了一起，开发中不使用|
|声明式事务控制|Spring将事务控制的代码封装，对外提供了Xml和注解配置方式，通过配置的方式完成事务的控制，可以达到事务控制与业务操作代码解耦合，开发中推荐使用|

## 相关类
Spring事务编程相关的类主要有如下三个
|事务控制相关类|解释|
|---|---|
|平台事务管理器 `PlatformTransactionManager`|一个接口标准，实现类都具备事务提交、回滚和获得事务对象的功能，不同持久层框架可能会有不同实现方案|
|事务定义 `TransactionDefinition`|封装事务的隔离级别、传播行为、过期时间等属性信息|
|事务状态 `TransactionStatus`|存储当前事务的状态信息，如果事务是否提交、是否回滚、是否有回滚点等|

### PlatformTransactionManager
平台事务管理器PlatformTransactionManager是Spring提供的封装事务具体操作的规范接口，封装了事务的提交和回滚方法：
```Java
public interface PlatformTransactionManager extends TransactionManager { 
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;

    void commit(TransactionStatus var1) throws TransactionException;

    void rollback(TransactionStatus var1) throws TransactionException;
}
```
不同的持久层框架事务操作的方式有可能不同，所以不同的持久层框架有可能会有不同的平台事务管理器实现。
* MyBatis作为持久层框架时：使用的平台事务管理器实现是DataSourceTransactionManager
* Hibernate作为持久层框架时：使用的平台事务管理器是HibernateTransactionManager


## Demo
导入Spring事务的相关的坐标，spring-jdbc坐标已经引入的spring-tx坐标：
```XML
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>
```

```XML
<!-- 配置目标类AccountServiceImpl -->
<bean id="accountService" class="com.itheima.service.impl.AccoutServiceImpl">
    <property name="accountMapper" ref="accountMapper"></property>
</bean>

<!--平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--Spring提供的事务通知-->
<tx:advice id="myAdvice" transaction-manager="transactionManager">
    <!-- 事务定义信息配置，即配置TransactionDefinition -->
    <tx:attributes>
        <tx:method name="transferMoney"/>
    </tx:attributes>
</tx:advice>

<!--使用advisor标签配置切面-->
<aop:config>
    <aop:advisor advice-ref="myAdvice" pointcut="execution(* com.itheima.service.impl.*.*(..))"/>
</aop:config>
```

## 事务定义信息
每个事务有很多特性，例如:隔离级别、只读状态、超时时间等，这些信息在开发时可以通过connection进行指定，也通过XML配置文件进行配置：
```XML
<!-- 父标签为<tx:advice> -->
<tx:attributes>
    <tx:method name="方法名称"
               isolation="隔离级别"
               propagation="传播行为"
               read-only="只读状态"
               timeout="超时时间"/>
</tx:attributes>
```

### name
name属性名称指定哪个方法要进行哪些事务的属性配置，此处需要区分的是切点表达式指定的方法与此处指定的方法的区别：
* 切点表达式，是过滤哪些方法可以进行事务增强
* 事务属性信息的name，是指定哪个方法要进行哪些事务属性的配置

方法名在配置时，也可以使用 `*` 进行模糊匹配，例如:
```XML
<tx:advice id="myAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--精确匹配transferMoney方法-->
        <tx:method name="transferMoney"/>
        <!--模糊匹配以Service结尾的方法-->
        <tx:method name="*Service"/>
        <!--模糊匹配以insert开头的方法-->
        <tx:method name="insert*"/>
        <!--模糊匹配以update开头的方法-->
        <tx:method name="update*"/>
        <!--模糊匹配任意方法，一般放到最后作为保底匹配-->
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

### isolation
isolation属性用于指定事务的隔离级别，事务并发存在三大问题: 脏读、不可重复读、幻读/虚读。可以通过设置事务的隔离级别来控制并发问题的发生与否。
|isolation属性|解释|
|---|---|
|DEFAULT|默认隔离级别，取决于当前数据库隔离级别，例如MySQL默认隔离级别是REPEATABLE_READ|
|READ_UNCOMMITTED|A事务可以读取到B事务尚未提交的事务记录，不能解决任何并发问题，安全性最低，性能最高|
|READ_COMMITTED|A事务只能读取到其他事务已经提交的记录，不能读取到未提交的记录。可以解决脏读问题，但是不能解决不可重复读和幻读|
|REPEATABLE_READ|A事务多次从数据库读取某条记录结果一致，可以解决不可重复读，不可以解决幻读|
|SERIALIZABLE|串行化，可以解决任何并发问题，安全性最高，但是性能最低|

### read-only
read-only属性用于设置当前的只读状态，如果是查询则设置为true，可以提高查询性能，如果是更新(增删改)操作则设置为false。

### timeout
timeout属性用于设置事务执行的超时时间，单位是秒，如果超过该时间限制但事务还没有完成，则自动回滚事务，不再继续执行。默认值是-1，即没有超时时间限制。

### propagation
propagation属性用于设置事务的传播行为，主要解决是"A方法调用B方法时，事务的传播方式"的问题。事务的传播行为有如下七种属性值可配置：
|事务传播行为|解释|
|---|---|
|REQUIRED(默认值)|A调用B，B需要事务，如果A有事务B就加入A的事务中，如果A没有事务，B就自己创建一个事务|
|REQUIRED_NEW|A调用B，B需要新事务，如果A有事务就挂起，B自己创建一个新的事务|
|SUPPORTS|A调用B，B有无事务无所谓，A有事务就加入到A事务中，A无事务B就以非事务方式执行|
|NOT_SUPPORTS|A调用B，B以无事务方式执行，A如有事务则挂起|
|NEVER|A调用B，B以无事务方式执行，A如有事务则抛出异常|
|MANDATORY|A调用B，B要加入A的事务中，如果A无事务就抛出异常|
|NESTED|A调用B，B创建一个新事务，A有事务就作为嵌套事务存在，A没事务就以创建的新事务执行|


## 原理
`<tx:advice>`标签使用的命名空间处理器是TxNamespaceHandler，内部注册的是解析器是TxAdviceBeanDefinitionParser：
```Java
// advice的解析器
this.registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
```

TxAdviceBeanDefinitionParser中指定了要注册的BeanDefinition：
```Java
// TxAdviceBeanDefinitionParser解析器对<tx:advice>标签进行解析时，会往容器中注册BeanDefinition
// 该BeanDefiniton的class为TransactionInterceptor的，beanName为<tx:advice>标签的id属性值
protected Class<?> getBeanClass(Element element) {
    return TransactionInterceptor.class;
}
```

![基于AOP的声明式事务控制+20231120130715](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8EAOP%E7%9A%84%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1%E6%8E%A7%E5%88%B6%2B20231120130715.png%2B2023-11-20-13-07-17)

![基于AOP的声明式事务控制+20231120131830](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8EAOP%E7%9A%84%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1%E6%8E%A7%E5%88%B6%2B20231120131830.png%2B2023-11-20-13-18-34)