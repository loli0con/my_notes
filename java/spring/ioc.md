# IOC
IoC全称Inversion of Control，直译为控制反转。

Ioc不是什么技术，而是一种设计思想，主要的描述是在软件开发的领域**对象的创建和管理**的问题。



## 引用
* https://www.liaoxuefeng.com/wiki/1252599548343744/1282381977747489
* https://www.cnblogs.com/xdp-gacl/p/4249939.html
* https://blog.csdn.net/zl1zl2zl3/article/details/106228798
* https://juejin.cn/post/6844903480298045448
* https://www.bilibili.com/video/BV1WE411d7Dv



## 什么是IOC
Ioc—Inversion of Control，即“**控制反转**”，不是什么技术，而是一种**设计思想**。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

如何理解好Ioc呢？理解好Ioc的关键是要明确“*谁控制谁，控制什么*，*为何是反转*（有反转就应该有正转了），*哪些方面反转了*”，那我们来深入分析一下。

### 控制
*谁控制谁，控制什么*：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是**IoC容器控制了对象**；控制什么？那就是主要**控制了外部资源获取**（不只是对象包括比如文件等）。

### 反转
*为何是反转，哪些方面反转了*：有反转就有正转，传统应用程序是由**我们自己在对象中主动控制去直接获取依赖对象**，也就是正转；而**反转则是由容器来帮忙创建及注入依赖对象**；为何是反转？因为**由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象**，所以是反转；哪些方面反转了？**依赖对象的获取方式被反转了**。

### Spring
所谓IoC，对于spring框架来说，就是**由spring来负责控制对象的生命周期和对象间的关系**。

Spring所倡导的开发方式：所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

使用Spring框架就是使用工厂模式来创建对象，由Spring容器帮我们管理所有的Java对象。

#### 容器
容器是一种为某种特定组件的运行提供必要支持的一个软件环境。通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。

#### 工厂
工厂负责创建对象的地方，并且把创建好的对象把对象放到容器中。在使用的时候，帮助我们从容器获取指定的对象。



## DI
DI—Dependency Injection，即“依赖注入”：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”，那我们来深入分析一下：
* 谁依赖谁：应用程序依赖于IoC容器
* 为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源
* 谁注入谁：IoC容器注入应用程序某个对象，应用程序依赖的对象
* 注入了什么：注入某个对象所需要的外部资源（包括对象、资源、常量数据）

### 相同点
IOC 和 DI 描述的都是同一件事情(对象的实例化以及维护对象与对象之间的依赖关系)，是同一个概念的不同角度描述。

由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。

### 不同点
IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是**通过DI（Dependency Injection，依赖注入）来实现的**。

IOC 是**站着对象的角度上**对象的实例化以及管理从程序员的手里交给了 IOC 容器，DI 是**站着容器的角度上**会把对象的依赖的其他对象注入到容器中。



## IOC的作用
### 存在的问题
如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。

因此，核心问题是：
* 谁负责创建组件？
* 谁负责根据依赖关系组装组件？
* 销毁时，如何按依赖顺序正确销毁？

### 解决方案
解决这一问题的核心方案就是IoC：
* IOC 解决了对象之间的耦合问题，将组件的创建+配置与组件的使用相分离；让对象脱离对依赖对象的维护，只需要随用随取，不需要关心依赖对象的任何过程。
* 同时 IOC 容器也方便管理容器内的所有 Bean 对象，所谓的 Bean 的生命周期。

### spring
在设计上，Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。这种无侵入的设计有以下好处：
* 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
* 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率。



## IOC的运用

### Bean
在 Spring 中，**构成应用程序主干**并**由Spring IoC容器管理的对象**称为bean。bean是一个由Spring IoC容器实例化、组装和管理的对象。

### BeanDefinition
BeanDefinition 描述了 Bean 的定义。我们定义的关于 Bean 的生命周期、销毁、初始化等操作总得有一个对象来承载，那么这个对象就是BeanDefinition。

这些被我们定义的Bean的元数据，会先加载到 BeanDefinition 上，然后SpringIoc容器通过 BeanDefinition 来生成一个Bean。从这个角度来说，BeanDefinition 和 Bean 的关系有点类似于类和对象的关系。

Spring容器，就是一个ApplicationContex对象，而启动Spring容器就是初始化这个对象，步骤可以分为两大部分：其一是通过各种手段获取到BeanDefinition，其二是通过获取到的BeanDefinition创建Bean。

### 配置的方式
Spring允许我们通过多种形式来定义Bean的元数据：
* 基于 **XML文件** 的配置方式
* 基于 **注解** 的配置方式
* 基于 **Java配置类** 的配置方式

这些元数据，都被放在SpringFactory里面，以一个Map的形式：Map<String,BeanDefinition>。

### 依赖注入的方式
常用的有两种方式：构造方法注入和setter注入



## xml文件 的 配置方式

### bean标签 - 简介
bean标签用于描述被IOC管理的对象，它有如下几个最常用的属性：
|属性|说明|
|---|---|
|id|容器中唯一的标识|
|name|还可以有多个名字，使用逗号，空格，分号隔开都可以|
|class|指定类全名，指定的是实现类，不是接口|
|scope|指定bean在容器中作用范围<br />singleton：默认值，表示这个是单例对象，整个容器中只会创建一个对象<br />prototype：这是多例对象，每次获取一个新的对象<br />request：用于请求域<br />session：用于会话域<br />application: 用于上下文域<br />globalsession：用于全局的会话，用在分布式开发中|
|init-method|创建对象时，执行的初始化的方法|
|destroy-method|销毁对象时，执行的方法；销毁的方法只在单例模式下起作用|
|lazy-init|是否使用延迟加载，默认是不使用；延迟加载只用于单例对象|

bean标签的内容（子标签）主要适用于描述该对象的依赖对象。

### bean标签 - 详解

#### 对象的创建
创建对象方式：
* 构造方法
  * 无参（默认）
  * 有参
* 静态工厂方法
* 实例工厂方法

##### 静态工厂方法
```java
/**
 * 静态工厂
 */
public class StaticBeanFactory {

    /**
     * 创建对象的静态方法
     */
    public static CustomerService getBean() {
        return new CustomerServiceImpl();
    }
}
```

```xml
<!--
使用静态工厂创建
class: 静态工厂类
factory-method：工厂中的获取对象的静态方法
id：对象的id  -->
<bean class="com.itheima.utils.StaticBeanFactory" factory-method="getBean" id="customerService"/>
```

##### 实例工厂方法
```java
/**
 * 实例工厂
 */
public class InstanceBeanFactory {

    /**
     * 不是静态方法
     */
    public CustomerService getBean() {
        return new CustomerServiceImpl();
    }
}
```

```xml
<!--
使用实例工厂来创建
    1 先创建实例工厂对象
    2 调用实例工厂的方法创建我们所需的对象
 -->
<bean class="com.itheima.utils.InstanceBeanFactory" id="instanceBeanFactory"/>
<!-- “上面的id” 等于 “下面的factory-bean” 
    
    factory-bean：实例工厂对象
    factory-method：工厂中创建对象的实例方法
-->
<bean id="customerService" factory-bean="instanceBeanFactory" factory-method="getBean"/>
```

#### 对象的生命周期
|scope取值|作用范围|生命周期|
|---|---|---|
|singleton 单例对象|容器一创建就创建这个对象，只要容器不销毁就一直存在|出生：容器创建就出生<br />活着：只要容器没有销毁就一直存在<br />死亡：容器关闭的时候|
|prototype 多例对象|每次获取对象就创建一个新的对象，使用完毕会被GC回收|出生：获取对象的时候<br />活着：使用过程中<br />死亡：由GC去回收|

#### 依赖的注入
```java
public class Customer {
    private int id;
    private String name;
    private boolean male;
    private Date birthday;
    // 省略无参构造方法、有参构造方法、getter和setter
}
```
##### 构造函数
|constructor-arg标签的属性|描述|
|---|---|
|index|构造方法参数的位置，从0开始|
|name|参数名|
|type|指定参数的类型|
|value|赋值简单类型=基本类型+String类型|
|ref|赋值引用类型|

```xml
<!-- 1.使用构造方法注入
子元素：constructor-arg 构造方法注入
    index：每几个位置，从0开始
    name：指定形参的名字
    type：指定参数的类型
    value：简单类型的值：8种基本类型+String类型
    ref：赋值引用类型，指定对象的id
 -->
<bean class="com.itheima.entity.Customer" id="customer">
    <constructor-arg name="id" value="100"/>
    <constructor-arg name="name" value="白骨精"/>
    <constructor-arg name="male" value="true"/>
    <constructor-arg name="birthday" ref="birthday" type="java.util.Date"/>
</bean>

<!-- 引用类型：现在的时间 -->
<bean class="java.util.Date" id="birthday"/>
```

###### c命名空间
```xml
<!-- 导入约束：xmlns:c="http://www.springframework.org/schema/c" -->

<!--C(构造：Constructor)命名空间-->
<bean id="user" class="com.kuang.pojo.User" c:name="狂神" c:age="18"/>
 ```

##### set方法
|\<property>的属性|描述|
|---|---|
|name|属性名|
|value|简单类型的值|
|ref|引用类型的值|


```xml
<!-- 2.使用set方法注入 -->
<bean class="com.itheima.entity.Customer" id="customer">
    <!-- 属性名：为id value为200 -->
    <property name="id" value="200"/>
    <property name="name" value="猪八戒"/>
    <property name="male" value="true"/>
    <property name="birthday" ref="birthday"/>
</bean>

<!-- 引用类型：现在的时间 -->
<bean class="java.util.Date" id="birthday"/>
```

###### p命名空间
```xml
<!-- 导入约束：xmlns:p="http://www.springframework.org/schema/p" -->
 
<!--P(属性：properties)命名空间, 本质上还是set注入-->
<bean id="user" class="com.kuang.pojo.User" p:name="狂神" p:age="18" p:birthday-ref="birthday"/>

<!-- 引用类型：现在的时间 -->
<bean class="java.util.Date" id="birthday"/>
```

##### 更多例子
```java
public class Student {
    private String name;  // String
    private Address address;  // 实体，Bean
    private String[] books;  // 数组
    private List<String> hobbys;  // List
    private Set<String> games;  // Set
    private Map<String,String> card;  // Map
    private String wife;  // null
    private Properties info;  //Properties

    // 省略getter和setter
}
```

```xml
<!-- 常量 -->
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
</bean>

<!-- Bean -->
<bean id="addr" class="com.kuang.pojo.Address">
    <property name="address" value="重庆"/>
</bean>
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
    <property name="address" ref="addr"/>
</bean>

<!-- 数组 -->
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
    <property name="address" ref="addr"/>
    <property name="books">
        <array>
            <value>西游记</value>
            <value>红楼梦</value>
            <value>水浒传</value>
        </array>
    </property>
</bean>

<!-- List -->
<property name="hobbys">
    <list>
        <value>听歌</value>
        <value>看电影</value>
        <value>爬山</value>
    </list>
</property>

<!-- Set -->
<property name="games">
    <set>
        <value>LOL</value>
        <value>BOB</value>
        <value>COC</value>
    </set>
</property>

<!-- Map -->
<property name="card">
    <map>
        <entry key="中国邮政" value="456456456465456"/>
        <entry key="建设" value="1456682255511"/>
    </map>
</property>

<!-- null -->
<property name="wife">
    <null/>
</property>

<!-- Properties -->
<property name="info">
    <props>
        <prop key="学号">20190604</prop>
        <prop key="性别">男</prop>
        <prop key="姓名">小明</prop>
    </props>
</property>
```

### 其他标签

#### alias标签
```xml
<!--设置别名：在获取Bean的时候可以使用别名获取-->
<alias name="userT" alias="userNew"/>
```

#### import标签
```xml
<import resource="路径/beans.xml"/>
```
1. 配置文件会先合并，后解析，也就是说，无论是命名空间还是配置的内容，都会合并处理（合并前的报错请忽略）。
2. 多个 Spring 配置文件最终会合并到一起，形成一个配置文件，因此这些配置中的 bean 都是可以互相引用的。

#### 读取配置文件
```xml
<!-- 读取配置文件(示例中为druid配置文件)，放在容器中，后面我们可以读取其中的属性值 -->
<context:property-placeholder location="classpath:druid.properties"/>

<!-- 使用配置文件中的属性 -->
<bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
</bean>
```


## 注解 的 配置方式
### 启用注解功能支持
需要在xml配置文件中做如下配置：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
    
    <!-- 开启属性注解支持 -->
    <context:annotation-config/>

    <!-- 扫描哪个包下所有的类（寻找注解），
    指定基包的名字（可以使用逗号分隔多个基包），
    使用context命名空间 -->
    <context:component-scan base-package="com.itheima"/>
    
</beans>
```
即两个配置：
1. 开启注解功能支持
2. 指定扫描组件的位置（去该位置下寻找Bean）


### 常用注解

#### @Component
Component注解放在类上，表示这个类会被创建，并将对象放在容器中。  
默认是类名首字母小写做为名字，也可以使用value属性来指定*该类的名字*（相当于配置文件中&lt;bean id="*该类的名字*" class="当前注解的类"/>）。

|创建对象的注解|说明|
|---|---|
|@Component|放在普通的类上|
|@Controller|放在控制器|
|@Service|放在业务对象|
|@Repository|放在持久层|

以上四个注解（@Component是另三个注解的父注解）的功能一样，只是语义上区别。

#### @Autowired
Autowired注解放在成员变量和成员方法上，表示将*相应的值*注入该属性（从容器中寻找合适的被依赖资源）。  
*相应的值*：
* 按类型匹配的方式从容器中去查找对应的值注入
* 如果有多个匹配的类型，按名字匹配的方式注入
* 如果找不到匹配的名字，就会抛出异常

@Autowired的属性required：默认为true，表示这个属性是必需的；如果为false，则匹配失败时，不抛出异常，将属性值为null。

#### @Qualifier
Autowired是根据类型自动装配的，加上@Qualifier则可以根据byName的方式自动装配：
* 位置: 放在成员变量或成员方法上
* 作用: 按名字匹配的方式注入
* 属性: value指定要注入的名字名

必须与@Autowired配置使用，不能单独使用

#### @Resource
Resource注解放在成员变量和成员方法上，表示从容器中寻找值并进行注入：
* 如有指定的name属性，按该属性进行byName方式查找装配；
* 否则按照默认的byName方式进行装配；
* 如果以上都不成功，则按byType的方式自动装配。
* 都不成功，则报异常。

#### @Value
注入一些简单类型，也可以[读取配置文件中值注入](https://www.cnblogs.com/binghe001/p/13216798.html)——结合@PropertySource等注解使用。
```java
public Class User{
    @Value("root")
    private username;

    @Value("true")
    private boolean sex;

    @Value("2011/11/11")
    private Date birthday;
}
```

#### @Scope
用在类上，用来指定当前对象在容器中是单例对象(singleton)还是多例对象(prototype)。

#### @Lazy
用在类上，用来指定当前对象是否延迟加载，在默认的情况(false)下，容器加载对象就加载了。如果为true表示延迟加载。

#### @PostConstruct
用在方法上，指定这是一个初始化的方法

#### @PreDestroy
用在方法上，指定这是对象销毁前执行的方法，只适用于单例对象。




## JavaConfig类 的 配置方式
JavaConfig 原来是 Spring 的一个子项目，它通过 Java 类的方式提供 Bean 的定义信息，在 Spring4 的版本， JavaConfig 已正式成为 Spring4 的核心功能 。

### 常用注解
#### @Configuration
放在类上，表示这是一个配置类（等价于beans.xml）。  
如果要读取配置类，要使用AnnotationConfigApplicationContext加载容器。

被@Configuration标注的类，也会被Spring容器接管并注册到容器当中，因为它也是一个@Component。

#### @ComponentScan
放在类上，指定要扫描的基包。
|@ComponentScan的属性|作用|
|---|---|
|basePackages|参数是一个字符数组，指定一个或多个基包的名字|
|value|同上|

#### @Bean
1. 注解用在方法上
2. 自动将方法的返回值加入到Spring容器中，方法名就是放在容器中的id。
3. 如果想指定与方法名不同的id，则使用name属性指定名字，相当于指定id(name属性的别名是value)。
4. 如果方法有参数，则参数传入的对象从容器中自动按类型匹配的方式去找。

#### @PropertySource
读取Java的属性文件(.properties)，value[]属性：指定一个或多个属性文件的名字。  
@PropertySource可以不用写classpath，因为注解默认从类路径下加载

#### @ImportResource
导入Spring的配置文件（beans.xml），让配置文件里面的内容生效

#### @Import
@Import是一个功能强大的注解，它通过指定一个或多个类对象，实现了如下的功能：
* 导入其他的配置类：在已有的Bean上，使用@Import注解，导入一个不在@ComponentScan扫描范围内的配置类
* 直接/间接将其他类导入到容器中
  * 直接指定其他的类的：在已有的Bean上，使用@Import注解，导入一个普通类。
  * 指定实现ImportSelector的类：在已有的Bean上，使用@Import注解，导入一个ImportSelector的实现类。该实现类在重写的selectImports方法中，返回的类全路径（以String[]的形式），返回值中的所有的类都会变成Bean。
  * 指定实现ImportBeanDefinitionRegistrar的类：已有的Bean上，使用@Import注解，导入一个ImportBeanDefinitionRegistrar的实现类，在重写的方法中注册BeanDefinition到Spring容器。



### 示例
#### 实体类
```java
@Component  //将这个类标注为Spring的一个组件，放到容器中！
public class Dog {
   public String name = "dog";
}
```

#### 配置类
```java
@Configuration  //代表这是一个配置类
@PropertySource("druid.properties")  // 加载配置文件
@Import(OtherConfig.class)  // 导入其他配置类
@ComponentScan("com.itheima")  // 扫描基包的中类，以支持注解配置功能
public class MyConfig {

    @Value("${jdbc.username}")  // 使用配置文件中的属性
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean //通过方法注册一个bean，这里的返回值就Bean的类型，方法名就是bean的id！
    public Dog dog(){
       return new Dog();
   }

    @Bean
    public DataSource createDataSource() {
        /* 省略部分代码 */
        return ds;
    }

    @Bean // 方法有参数ds，按类型DataSource匹配的方式从容器中去查找
    public JdbcTemplate createJdbcTemplate(DataSource ds) {
        return new JdbcTemplate(ds);
    }
}
```

#### 测试类
```java
@Test
public void test(){
   ApplicationContext applicationContext =
           new AnnotationConfigApplicationContext(MyConfig.class);
   Dog dog = (Dog) applicationContext.getBean("dog");
   System.out.println(dog.name);
}
```


## 容器API
![ioc+20210728140220](https://i.loli.net/2021/07/28/5CezFtbH4GywS6X.png)
### 示例
```java
/** 
创建：
    ClassPathXmlApplicationContext
    方式一：类路径配置文件创建容器

    FileSystemXmlApplicationContext
    方式二：本地配置文件方式创建容器

    AnnotationConfigApplicationContext
    方式三：注解的方式创建容器
*/
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

ApplicationContext context = new FileSystemXmlApplicationContext("xxx路径/applicationContext.xml");

ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);

//2.从容器中获取对象
CustomerService customerService = (CustomerService) context.getBean("customerService");
//3.调用对象的方法
customerService.doCustomer();
//4.关闭容器
context.close();
```