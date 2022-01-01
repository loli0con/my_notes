# Spring Data Jpa
Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据库的访问和操作。

Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现，在实际的工作工程中，推荐使用Spring Data JPA + ORM（如：hibernate）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。

## 先导知识
### ORM
ORM（Object-Relational Mapping）表示对象关系映射。在面向对象的软件开发中，通过ORM，就可以把对象映射到关系型数据库中。只要有一套程序能够做到建立对象与数据库的关联，操作对象就可以直接操作数据库数据，就可以说这套程序实现了ORM对象关系映射。

### JPA规范
JPA的全称是Java Persistence API，即Java持久化API，是SUN公司推出的一套基于ORM的规范，内部是由一系列的接口和抽象类构成。

JPA通过JDK 5.0注解描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

JPA 包括以下三方面的内容：
1. 一套 API 标准，在 javax.persistence 的包下面，用来操作实体对象，执行 CRUD 操作，框架在后台替代我们完成所有的事情，开发者从繁琐的 JDBC 和 SQL 代码中解脱出来。
2. 面向对象的查询语言：Java Persistence Query Language（JPQL），这是持久化操作中很重要的一个方面，通过面向对象而非面向数据库的查询语言查询数据，避免程序的 SQL 语句紧密耦合。
3. ORM（Object/Relational Metadata）元数据的映射，JPA 支持 XML 和 JDK 5.0 注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中。


### Hibernate
Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，它将POJO与数据库表建立映射关系，是一个全自动的orm框架，hibernate可以自动生成SQL语句，自动执行，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。

### 总结
JPA是一套规范，内部是有接口和抽象类组成的。hibernate是一套成熟的ORM框架，而且Hibernate实现了JPA规范，所以也可以称hibernate为JPA的一种实现方式，我们使用JPA的API编程，意味着站在更高的角度上看待问题（面向接口编程）

![jap+20211122164602](https://raw.githubusercontent.com/loli0con/picgo/master/images/jap%2B20211122164602.png%2B2021-11-22-16-46-03)

Spring Data JPA是Spring提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案。



## 配置实体类
* @Entity
  * 作用：指定当前类是实体类。
* @Table
  * 作用：指定实体类和表之间的对应关系。
  * 属性：
    * name：指定数据库表的名称。
* @Id
  * 作用：指定当前字段是主键。
* @GeneratedValue
  * 作用：指定主键的生成方式。
  * 属性：
    * strategy：指定主键生成策略，JPA提供的四种标准用法为TABLE、SEQUENCE、IDENTITY、AUTO。
      * IDENTITY：主键由数据库自动生成（主要是自动增长型）。
      * SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。
      * AUTO：主键由程序控制。
      * TABLE：使用一个特定的数据库表格来保存主键。
* @Column
  * 作用：指定实体类属性和数据库表之间的对应关系。
  * 属性：
    * name：指定数据库表的列名称。
    * unique：是否唯一。
    * nullable：是否可以为空
    * inserttable：是否可以插入。
    * updateable：是否可以更新。
    * columnDefinition: 定义建表时创建此列的DDL。
    * secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字。

### demo
```java
@Entity //声明实体类
@Table(name="cst_customer") //建立实体类和表的映射关系
public class Customer {

    @Id//声明当前私有属性为主键
    @GeneratedValue(strategy=GenerationType.IDENTITY) //配置主键的生成策略
    @Column(name="cust_id") //指定和表中cust_id字段的映射关系
    private Long custId;

    @Column(name="cust_name") //指定和表中cust_name字段的映射关系
    private String custName;

    @Column(name="cust_source")//指定和表中cust_source字段的映射关系
    private String custSource;

    @Column(name="cust_industry")//指定和表中cust_industry字段的映射关系
    private String custIndustry;

    @Column(name="cust_level")//指定和表中cust_level字段的映射关系
    private String custLevel;

    @Column(name="cust_address")//指定和表中cust_address字段的映射关系
    private String custAddress;

    @Column(name="cust_phone")//指定和表中cust_phone字段的映射关系
    private String custPhone;

    // 省略getter和setter
}
```

## 配置实现类
在java工程的src路径下名为META-INF的文件夹，在此文件夹下一个名为persistence.xml的配置文件：
```xml
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence  
    http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
    version="2.0">
    <!--配置持久化单元 
        name：持久化单元名称 
        transaction-type：事务类型
            RESOURCE_LOCAL：本地事务管理 
            JTA：分布式事务管理 -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!-- 配置JPA规范的服务提供商，示例中为hibernate -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <!-- 数据库驱动 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
            <!-- 数据库地址 -->
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/db" />
            <!-- 数据库用户名 -->
            <property name="javax.persistence.jdbc.user" value="root" />
            <!-- 数据库密码 -->
            <property name="javax.persistence.jdbc.password" value="root" />

            <!--jpa提供者的可选配置：我们的JPA规范的提供者为hibernate，所以jpa的核心配置中兼容hibernate的配置 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```

## JPA API

### Persistence
Persistence对象主要作用是用于获取EntityManagerFactory对象的。通过调用该类的createEntityManagerFactory静态方法，根据配置文件中持久化单元名称创建EntityManagerFactory。
```java
//1. 创建 EntitymanagerFactory
String unitName = "myJpa";
EntityManagerFactory factory= Persistence.createEntityManagerFactory(unitName);
```

### EntityManagerFactory
EntityManagerFactory 接口主要用来创建 EntityManager 实例。
```java
//创建实体管理类
EntityManager em = factory.createEntityManager();
```

#### JPAUtil
由于EntityManagerFactory是一个线程安全的对象（即多个线程访问同一个EntityManagerFactory对象不会有线程安全问题），并且EntityManagerFactory的创建极其浪费资源，所以在使用JPA编程时，我们可以对EntityManagerFactory的创建进行优化，只需要做到一个工程只存在一个EntityManagerFactory即可。

```java
// 单例模式 - 饿汉式
public final class JPAUtil {
	// JPA的实体管理器工厂：相当于Hibernate的SessionFactory
	private static EntityManagerFactory em;
	// 使用静态代码块赋值
	static {
		// 注意：该方法参数必须和persistence.xml中persistence-unit标签name属性取值一致
		em = Persistence.createEntityManagerFactory("myJpa");
	}

	/**
	 * 使用管理器工厂生产一个管理器对象
	 * 
	 * @return
	 */
	public static EntityManager getEntityManager() {
		return em.createEntityManager();
	}
}
```

### EntityManager
在 JPA 规范中, EntityManager是完成持久化操作的核心对象。实体类作为普通java对象，只有在调用 EntityManager将其持久化后才会变成持久化对象。EntityManager对象在一组实体类与底层数据源之间进行 O/R 映射的管理。它可以用来管理和更新 Entity Bean，椐主键查找 Entity Bean，还可以通过JPQL语句查询实体。

我们可以通过调用EntityManager的方法完成获取事务，以及持久化数据库的操作，方法说明如下：
* getTransaction：获取事务对象
* persist：保存操作
* merge：更新操作
* remove：删除操作
* find/getReference(延迟加载)：根据id查询

### EntityTransaction
在 JPA 规范中，EntityTransaction是完成事务操作的核心对象，对于EntityTransaction在我们的java代码中承接的功能比较简单，方法如下：
* begin：开启事务
* commit：提交事务
* rollback：回滚事务


### JPQL
JPQL全称Java Persistence Query Language。Java持久化查询语言(JPQL)是一种可移植的查询语言，旨在以面向对象表达式语言的表达式，将SQL语法和简单查询语义绑定在一起使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上的SQL。

其特征与原生SQL语句类似，并且完全面向对象，通过类名和属性访问，而不是表名和表的属性。

### demo
```java
@Test
public void findAll() {
    EntityManager em = null;
    EntityTransaction tx = null;
    try {
        //获取实体管理对象
        em = JPAUtil.getEntityManager(); //工厂模式 + 单例模式
        //获取事务对象
        tx = em.getTransaction();
        //开启事务
        tx.begin();

        // 创建query对象
        String jpql = "from Customer where custName like ? order by custId desc";
        // String jpql = "select count(custId) from Customer";
        Query query = em.createQuery(jpql);
        //对占位符赋值，从1开始
        query.setParameter(1, "%猪%");

        //分页
        //起始索引
        query.setFirstResult(0);
        //每页显示条数
        query.setMaxResults(2);

        // 查询并得到返回结果
        List list = query.getResultList(); // 得到集合返回类型
        for (Object object : list) {
            System.out.println(object);
        }

        // Object count = query.getSingleResult(); // 得到对象返回类型

        //提交事务
        tx.commit();
    } catch (Exception e) {
        // 回滚事务
        tx.rollback();
        e.printStackTrace();
    } finally {
        // 释放资源
        em.close();
    }
}
```



## Spring Data JPA

### 配置
spring的配置文件（如果是SpringBoot则不需要配这些）。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa" xmlns:task="http://www.springframework.org/schema/task"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/data/jpa 
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
    
    <!-- 1.dataSource 配置数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/jpa" />
        <property name="user" value="root" />
        <property name="password" value="root" />
    </bean>
    
    <!-- 2.配置entityManagerFactory -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="packagesToScan" value="fun.pigyun.entity" />
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider" />
        </property>
        <!--JPA的供应商适配器-->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <property name="generateDdl" value="false" />
                <property name="database" value="MYSQL" />
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect" />
                <property name="showSql" value="true" />
            </bean>
        </property>
        <property name="jpaDialect">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
        </property>
    </bean>
    
    
    <!-- 3.事务管理器-->
    <!-- JPA事务管理器  -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory" />
    </bean>
    
    <!-- 整合spring data jpa-->
    <jpa:repositories base-package="fun.pigyun.dao"
        transaction-manager-ref="transactionManager"
        entity-manager-factory-ref="entityManagerFactory"></jpa:repositories>
        
    <!-- 4.txAdvice-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    
    <!-- 5.aop-->
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* fun.pigyun.service.*.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
    </aop:config>
    
    <context:component-scan base-package="fun.pigyun"></context:component-scan>
        
    <!--组装其它 配置文件-->
</beans>
```

### Dao层接口
Spring Data JPA是spring提供的一款对于数据访问层（Dao层）的框架，使用Spring Data JPA，只需要按照框架的规范提供dao接口，不需要实现类就可以完成数据库的增删改查、分页查询等方法的定义，极大的简化了我们的开发过程。
```java
/**
 * JpaRepository<实体类类型，主键类型>：用来完成基本CRUD操作
 * JpaSpecificationExecutor<实体类类型>：用于复杂查询（分页等查询操作）
 */
public interface CustomerDao extends JpaRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
}
```

### API
在继承JpaRepository和JpaRepository接口后，我们就可以使用接口中定义的方法进行查询。
![jap+20211122173934](https://raw.githubusercontent.com/loli0con/picgo/master/images/jap%2B20211122173934.png%2B2021-11-22-17-39-36)

![jap+20211122173955](https://raw.githubusercontent.com/loli0con/picgo/master/images/jap%2B20211122173955.png%2B2021-11-22-17-39-56)

### JPQL
使用Spring Data JPA提供的查询方法已经可以解决大部分的应用场景，但是对于某些业务来说，我们还需要灵活的构造查询条件，这时就可以使用@Query注解，结合JPQL的语句方式完成查询。

#### demo
@Query注解的使用非常简单，只需在方法上面标注该注解，同时提供一个JPQL查询语句即可。

```java
public interface CustomerDao extends JpaRepository<Customer, Long>,JpaSpecificationExecutor<Customer> {    
    //@Query 使用jpql的方式查询。
    @Query(value="from Customer")
    public List<Customer> findAllCustomer();
    
    //@Query 使用jpql的方式查询。?1代表参数的占位符，其中1对应方法中的参数索引
    @Query(value="from Customer where custName = ?1")
    public Customer findCustomer(String custName);

    @Query(value="update Customer set custName = ?1 where custId = ?2")
    @Modifying // 将该操作标识为修改查询，这样框架最终会生成一个更新的操作
    public void updateCustomer(String custName,Long custId);
    
    /**
     * nativeQuery : 使用本地sql的方式查询
     */
    @Query(value="select * from cst_customer",nativeQuery=true)
    public void findSql();
}
```

### 方法命名规则查询
方法命名规则查询就是根据方法的名字，就能创建查询。只需要按照Spring Data JPA提供的方法命名规则定义方法的名称，就可以完成查询工作。Spring Data JPA在程序执行的时候会根据方法名称进行解析，并自动生成查询语句进行查询。

按照Spring Data JPA 定义的规则，查询方法以findBy开头，涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性首字母需大写。框架在进行方法名解析时，会先把方法名多余的前缀截取掉，然后对剩下部分进行解析。

具体的关键字，使用方法和生产成SQL如下表所示：
| Keyword           | Sample                                  | JPQL                                                           |
| ----------------- | --------------------------------------- | -------------------------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2                   |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2                    |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                       |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2                          |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                                             |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age <= ?1                                            |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                                             |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                                            |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                                       |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                                       |
| IsNull            | findByAgeIsNull                         | … where x.age is null                                          |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                                         |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                                    |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1                                |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %)  |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %)     |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc                    |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                                       |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                                            |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                                        |
| TRUE              | findByActiveTrue()                      | … where x.active = true                                        |
| FALSE             | findByActiveFalse()                     | … where x.active = false                                       |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)                          |


### 动态SQL
有时我们在查询某个实体的时候，给定的条件是不固定的，这时就需要动态构建相应的查询语句，在Spring Data JPA中可以通过`JpaSpecificationExecutor`接口查询。相比JPQL，其优势是类型安全，更加的面向对象。

```java
import java.util.List;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;

/**
 *	JpaSpecificationExecutor中定义的方法
 **/
 public interface JpaSpecificationExecutor<T> {
       //根据条件查询一个对象
     T findOne(Specification<T> spec);	
       //根据条件查询集合
     List<T> findAll(Specification<T> spec);
       //根据条件分页查询
     Page<T> findAll(Specification<T> spec, Pageable pageable);
       //排序查询查询
     List<T> findAll(Specification<T> spec, Sort sort);
       //统计查询
     long count(Specification<T> spec);
}
```

对于JpaSpecificationExecutor，这个接口基本是围绕着Specification接口来定义的。我们可以简单的理解为，Specification构造的就是查询条件。  
Specification接口中只定义了如下一个方法：
```java
//构造查询条件
/**
*	root	：Root接口，代表查询的根对象，可以通过root获取实体中的属性
*	query	：代表一个顶层查询对象，用来自定义查询
*	cb		：用来构建查询，此对象里有很多条件方法
**/
public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
```

#### demo
```java
public void testSpecifications() {
    //使用匿名内部类的方式，创建一个Specification的实现类，并实现toPredicate方法
    Specification <Customer> spec = new Specification<Customer>() {
        public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
            //cb:构建查询，添加查询方式
            //  like表示模糊匹配，还有更多的方法：
            //  equle、gt、lt、ge、le、notEqule、like、notLike
            //root:从实体Customer对象中按照custName属性进行查询
            return cb.like(root.get("custName").as(String.class), "%猪%");
        }
    };

    /**
    * 构造分页参数
    * Pageable: 接口
    * PageRequest实现了Pageable接口，调用构造方法的形式构造
    * 	第一个参数：页码（从0开始）
    * 	第二个参数：每页查询条数
    */
    Pageable pageable = new PageRequest(0, 5);

    /**
    * 分页查询，封装为Spring Data Jpa 内部的page bean
    * 此重载的findAll方法为分页方法需要两个参数
    * 	第一个参数：查询条件Specification
    * 	第二个参数：分页参数
    */
    Page<Customer> page = customerDao.findAll(spec, pageable);
}
```

对于Spring Data JPA中的分页查询，是其内部自动实现的封装过程，返回的是一个Spring Data JPA提供的pageBean对象。其中的方法说明如下：
* int getTotalPages()：获取总页数
* long getTotalElements()：获取总记录数
* List\<T> getContent()：获取列表数据



## 多表设计

### 一对多

#### 注解
* @OneToMany:
  * 作用：建立一对多的关系映射
  * 属性：
    * targetEntityClass：指定多的多方的类的字节码
    * mappedBy：指定从表实体类中引用主表对象的名称。
    * cascade：指定要使用的级联操作
    * fetch：指定是否采用延迟加载，可选的配置值为：
      * FetchType.EAGER：立即加载
      * FetchType.LAZY：延迟加载
    * orphanRemoval：是否使用孤儿删除
* @ManyToOne
  * 作用：建立多对一的关系
  * 属性：
    * targetEntityClass：指定一的一方实体类字节码
    * cascade：指定要使用的级联操作，可选的配置值为：
      * CascadeType.MERGE：级联更新
      * CascadeType.PERSIST：级联保存
      * CascadeType.REFRESH：级联刷新
      * CascadeType.REMOVE：级联删除
      * CascadeType.ALL：包含所有
    * fetch：指定是否采用延迟加载
    * optional：关联是否可选。如果设置为false，则必须始终存在非空关系。
* @JoinColumn
  * 作用：用于定义主键字段和外键字段的对应关系。
  * 属性：
    * name：指定本表中的外键字段的名称
    * referencedColumnName：指定引用主表的主键字段名称
    * unique：是否唯一。默认值不唯一nullable：是否允许为空。默认值允许。
    * insertable：是否允许插入。默认值允许。
    * updatable：是否允许更新。默认值允许。
    * columnDefinition：列的定义信息。

#### demo

#### “一”
```java
**
 * 客户的实体类
 * 明确使用的注解都是JPA规范的
 * 所以导包都要导入javax.persistence包下的
 */
@Entity//表示当前类是一个实体类
@Table(name="cst_customer")//建立当前实体类和表之间的对应关系
public class Customer implements Serializable {
    
    @Id//表明当前私有属性是主键
    @GeneratedValue(strategy=GenerationType.IDENTITY)//指定主键的生成策略
    @Column(name="cust_id")//指定和数据库表中的cust_id列对应
    private Long custId;
    
    //配置客户和联系人的一对多关系
      // @OneToMany(targetEntity=LinkMan.class)
    // @JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
    // 上面两行代码会让主表去维护外键关系，会导致重复执行update语句的bug。因此主表，也就是“一”的一方，要放弃对外键关系的维护：
    @OneToMany(mappedBy="customer",cascade=CascadeType.ALL)
    private Set<LinkMan> linkmans = new HashSet<LinkMan>(0);
}
```
#### “多”
```java
/**
 * 联系人的实体类（数据模型）
 */
@Entity
@Table(name="cst_linkman")
public class LinkMan implements Serializable {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="lkm_id")
    private Long lkmId;

    //多对一关系映射：多个联系人对应客户
    @ManyToOne(targetEntity=Customer.class)
    @JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
    private Customer customer;//用它的主键，对应联系人表中的外键
}
```

### 多对多

#### 注解
* @ManyToMany
  * 作用：用于映射多对多关系
  * 属性：
    * cascade：配置级联操作。
    * fetch：配置是否采用延迟加载。
    * targetEntity：配置目标的实体类。映射多对多的时候不用写。
* @JoinTable
  * 作用：针对中间表的配置
  * 属性：
    * name：配置中间表的名称
    * joinColumns：中间表的外键字段关联当前实体类所对应表的主键字段			  			
    * inverseJoinColumn：中间表的外键字段关联对方表的主键字段
* @JoinColumn
  * 作用：用于定义主键字段和外键字段的对应关系。
  * 属性：
    * name：指定外键字段的名称
    * referencedColumnName：指定引用主表的主键字段名称
    * unique：是否唯一。默认值不唯一
    * nullable：是否允许为空。默认值允许。
    * insertable：是否允许插入。默认值允许。
    * updatable：是否允许更新。默认值允许。
    * columnDefinition：列的定义信息。

#### 主体
```java
@Entity
@Table(name = "sys_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="user_id")
    private Long userId;

    /**
     * 配置用户到角色的多对多关系
     *      配置多对多的映射关系
     *          1.声明表关系的配置
     *              @ManyToMany(targetEntity = Role.class)  //多对多
     *                  targetEntity：代表对方的实体类字节码
     *          2.配置中间表（包含两个外键）
     *                @JoinTable
     *                  name : 中间表的名称
     *                  joinColumns：配置当前对象在中间表的外键
     *                      @JoinColumn的数组
     *                          name：外键名
     *                          referencedColumnName：参照的主表的主键名
     *                  inverseJoinColumns：配置对方对象在中间表的外键
     */
    @ManyToMany(targetEntity = Role.class)
    @JoinTable(name = "sys_user_role",
        //joinColumns,当前对象在中间表中的外键
        joinColumns = {@JoinColumn(name = "sys_user_id",referencedColumnName = "user_id")},
        //inverseJoinColumns，对方对象在中间表的外键
        inverseJoinColumns = {@JoinColumn(name = "sys_role_id",referencedColumnName = "role_id")}
    )
    private Set<Role> roles = new HashSet<>();
}
```

#### 客体
在多对多（保存）中，如果双向都设置关系，意味着双方都维护中间表，都会往中间表插入数据，中间表的2个字段又作为联合主键，所以报错，主键重复，解决保存失败的问题：只需要在任意一方放弃对中间表的维护权即可，推荐在被动的一方放弃。

```java
@Entity
@Table(name = "sys_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;

    //配置多对多
    @ManyToMany(mappedBy = "roles")  //配置多表关系，放弃对中间表的维护权
    private Set<User> users = new HashSet<>(0);
}
```

## 多表查询

### 对象导航查询
对象图导航检索方式是根据已经加载的对象，导航到他的关联对象。它利用类与类之间的关系来检索对象。例如：我们通过ID查询方式查出一个客户，可以调用Customer类中的getLinkMans()方法来获取该客户的所有联系人。对象导航查询的使用要求是：两个对象之间必须存在关联关系。

### Specification
```java
/**
* Specification的多表查询
*/
@Test
public void testFind() {
    Specification<LinkMan> spec = new Specification<LinkMan>() {
        public Predicate toPredicate(Root<LinkMan> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
            //Join代表链接查询，通过root对象获取
            //创建的过程中，第一个参数为关联对象的属性名称，第二个参数为连接查询的方式（left、inner、right）
            //JoinType.LEFT：左外连接，JoinType.INNER：内连接，JoinType.RIGHT：右外连接
            Join<LinkMan, Customer> join = root.join("customer",JoinType.INNER);
            return cb.like(join.get("custName").as(String.class),"%猪%");
        }
    };
    List<LinkMan> list = linkManDao.findAll(spec);
    for (LinkMan linkMan : list) {
        System.out.println(linkMan);
    }
}
```

## 核心梳理

### Spring Data JPA 的主要类及结构图
![jap+20211202202255](https://raw.githubusercontent.com/loli0con/picgo/master/images/jap%2B20211202202255.png%2B2021-12-02-20-22-57)

七个大 Repository 接口：
* Repository（org.springframework.data.repository）；
* CrudRepository（org.springframework.data.repository）；
* PagingAndSortingRepository（org.springframework.data.repository）；
* JpaRepository（org.springframework.data.jpa.repository）；
* QueryByExampleExecutor（org.springframework.data.repository.query）；
* JpaSpecificationExecutor（org.springframework.data.jpa.repository）；
* QueryDslPredicateExecutor（org.springframework.data.querydsl）。

两大 Repository 实现类：
* SimpleJpaRepository（org.springframework.data.jpa.repository.support）；
* QueryDslJpaRepository（org.springframework.data.jpa.repository.support）。

真正的 JPA 的底层封装类：
* EntityManager（javax.persistence）；
* EntityManagerImpl（org.hibernate.jpa.internal）。

### 

## 参考阅读
* [持久层框架JPA与Mybatis该如何选型](https://cloud.tencent.com/developer/article/1702931)
* [@OneToMany的mappedBy属性-详解](https://www.cnblogs.com/chiangchou/p/mappedby.html)