# IoC
IoC全称Inversion of Control，直译为控制反转。

IoC不是一门技术，而是一种设计思想，是一个重要的面向对象编程法则，能够指导我们如何设计出松耦合、更优良的程序。主要的描述是在软件开发的领域**对象的创建和管理**的问题。

Spring 通过 IoC 容器来管理所有 Java 对象的实例化和初始化，控制对象与对象之间的依赖关系。我们将由 IoC 容器管理的 Java 对象称为 Spring Bean，它与使用关键字 new 创建的 Java 对象没有任何区别。



## IoC的内涵
如何理解好Ioc呢？理解好Ioc的关键是要明确“*谁控制谁，控制什么*，*为何是反转*（有反转就应该有正转了），*哪些方面反转了*”，那我们来深入分析一下。

### 控制
*谁控制谁，控制什么*：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是**IoC容器控制了对象**；控制什么？那就是主要**控制了外部资源获取**（不只是对象，还包括文件等资源。⚠️软件系统由对象组成，控制对象即控制一切）。

### 反转
*为何是反转，哪些方面反转了*：有反转就有正转，传统应用程序是由**我们自己在对象中直接主动地控制/获取依赖对象**，也就是正转；而**反转则是由容器来帮忙创建及注入依赖对象**；为何是反转？因为**由容器帮我们查找及注入依赖对象，而对象只是被动的接受依赖对象**，所以是反转；哪些方面反转了？**依赖对象的获取方式被反转了**。



## IOC的作用
* IOC 解决了对象之间的耦合问题，将组件的创建+配置与组件的使用相分离；让对象脱离对依赖对象的维护，只需要随用随取，不需要关心依赖对象的任何过程。
* 同时 IOC 容器也方便管理容器内的所有 Bean 对象（Bean的生命周期）。

### Spring
所谓IoC，对于Spring框架来说，就是**由Spring来负责控制对象的生命周期和对象间的关系**。

Spring所倡导的开发方式：所有的类都会在Spring容器中登记，告诉Spring‘你是个什么东西，你需要什么东西’，然后Spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由Spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是Spring。对于某个具体的对象而言，以前是它控制其他依赖对象，现在是所有对象都被Spring控制，所以这叫控制反转。

在设计上，Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。这种无侵入的设计有以下好处：
* 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
* 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率。

使用Spring框架就是使用工厂模式来创建对象，由Spring容器帮我们管理所有的Java对象。

#### 容器
容器是一种为某种特定组件的运行提供必要支持的一个软件环境。通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。

#### 工厂
工厂是负责创建对象的地方，并且把创建好的对象放到容器中。在使用的时候，帮助我们从容器获取指定的对象。



## IOC的运用

### Bean
在 Spring 中，**构成应用程序主干**并**由Spring IoC容器管理的对象**称为bean。bean是一个由Spring IoC容器实例化、组装和管理的对象。

Bean管理说的是：Bean对象的创建，以及Bean对象中属性的赋值（或者叫做Bean对象之间关系的维护）。

### BeanDefinition
BeanDefinition 描述了 Bean 的定义。我们定义的关于 Bean 的生命周期、销毁、初始化等操作数据总得有一个对象来承载，这个对象就是BeanDefinition。

这些被我们定义的Bean的元数据，会先加载到 BeanDefinition 上，然后SpringIoc容器通过 BeanDefinition 来生成一个Bean。从这个角度来说，BeanDefinition 和 Bean 的关系有点类似于类和对象的关系。

Spring容器，就是一个ApplicationContex对象，而启动Spring容器就是初始化这个对象，步骤可以分为两大部分：其一是通过各种手段获取到BeanDefinition，其二是通过获取到的BeanDefinition创建Bean。

Spring容器加载到Bean类时，会把这个类的描述信息，以包名加类名的方式存到beanDefinitionMap中`Map<String,BeanDefinition>`。其中String是Key，默认是类名首字母小写；BeanDefinition存的是类的定义(描述信息)，通常叫BeanDefinition接口为bean的定义对象。

### BeanFactory
BeanFactory是 IoC 容器的基本实现，是 Spring 内部使用的接口。ApplicationContext是BeanFactory 的子接口，提供了更多高级特性。面向 Spring 的使用者，几乎所有场合都使用 ApplicationContext 而不是底层的 BeanFactory。

ApplicationContext的主要实现类：
![ioc+20230926095535](https://raw.githubusercontent.com/loli0con/picgo/master/images/ioc%2B20230926095535.png%2B2023-09-26-09-55-36)

|类型名|简介|
|---|---|
|ClassPathXmlApplicationContext|通过读取类路径下的 XML 格式的配置文件创建 IOC 容器对象|
|FileSystemXmlApplicationContext|通过文件系统路径读取 XML 格式的配置文件创建 IOC 容器对象|
|ConfigurableApplicationContext|ApplicationContext 的子接口，包含一些扩展方法 refresh() 和 close() ，让 ApplicationContext 具有启动、关闭和刷新上下文的能力。|
|WebApplicationContext|专门为 Web 应用准备，基于 Web 环境创建 IOC 容器对象，并将对象引入存入 ServletContext 域中。|

### 配置的方式
Spring允许我们通过多种形式来定义Bean的元数据：
* [基于 **XML文件** 的配置方式](#xml文件-的-配置方式)
* [基于 **注解** 的配置方式](#注解-的-配置方式)
* [基于 **Java配置类** 的配置方式](#java配置类-的-配置方式)





## XML文件 的 配置方式

### bean标签
bean标签用于描述被IOC管理的对象，它有如下几个最常用的属性：
|属性|说明|
|---|---|
|id|容器中唯一的标识|
|name|还可以有多个名字，使用逗号，空格，分号隔开都可以|
|class|指定类全名，指定的是实现类，不是接口|
|scope| 指定bean在容器中作用范围<br />singleton：默认值，表示这个是单例对象，整个容器中只会创建一个对象；容器一创建就创建这个对象，只要容器不销毁就一直存在<br />prototype：这是多例对象，每次获取对象就创建一个新的对象，使用完毕会被GC回收<br />request：用于请求域<br />session：用于会话域<br />application: 用于上下文域<br />globalsession：用于全局的会话，用在分布式开发中|
|init-method| 创建对象时，执行的初始化的方法|
|destroy-method|销毁对象时，执行的方法；销毁的方法只在单例模式下起作用|
|lazy-init|是否使用延迟加载，默认是不使用；延迟加载只用于单例对象|

bean标签的内容（子标签）主要适用于描述该对象的依赖对象。

#### bean生命周期
1. bean对象创建（调用无参构造器）
2. 给bean对象设置属性
3. bean的后置处理器（初始化之前）
4. bean对象初始化（需在配置bean时指定初始化方法init-method）
5. bean的后置处理器（初始化之后）
6. bean对象就绪可以使用
7. bean对象销毁（需在配置bean时指定销毁方法destroy-method）
8. IOC容器关闭

##### bean的后置处理器
bean的后置处理器会在生命周期的初始化前后添加额外的操作，需要实现BeanPostProcessor接口，且配置到IOC容器中，需要注意的是，bean后置处理器不是单独针对某一个bean生效，而是针对IOC容器中所有bean都会执行

创建bean的后置处理器：
```Java
package com.atguigu.spring6.process;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("bef " + beanName + " = " + bean);
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("aft " + beanName + " = " + bean);
        return bean;
    }
}
```

在IOC容器中配置后置处理器：
```XML
<!-- bean的后置处理器要放入IOC容器才能生效 -->
<bean id="myBeanProcessor" class="com.atguigu.spring6.process.MyBeanProcessor"/>
```

#### 对象的创建
创建对象方式：
* 构造方法
  * 无参（默认，setter注入）
  * 有参（构造器注入）
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

#### 依赖注入
```java
public class Customer {
    private int id;
    private String name;
    private boolean male;
    private Date birthday;
    // 省略无参构造方法、有参构造方法、getter和setter
}
```

##### 构造器注入
|constructor-arg标签的属性|描述|
|---|---|
|index|构造方法参数的位置，从0开始|
|name|参数名|
|type|指定参数的类型|
|value|赋值简单类型（基本类型和String类型）|
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

##### setter注入
|property标签的属性|描述|
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

##### 更多例子（不同的数据类型）
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
<!-- 外部bean -->
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
    <property name="address" ref="addr"/>
</bean>
<!-- 内部bean -->
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
    <property name="address">
        <!-- 在一个bean中再声明一个bean就是内部bean -->
        <!-- 内部bean只能用于给属性赋值，不能在外部通过IOC容器获取，因此可以省略id属性 -->
        <bean id="student" class="com.kuang.pojo.Student">
            <property name="name" value="小明"/>
            <property name="address" ref="addr"/>
        </bean>
    </property>
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
        <entry>
            <key>
                <value>工商<value>
            </key>
            <value>1456682255512<value>
        </entry>
        <entry>
            <key>
                <value>工商<value>
            </key>
            <ref bean="xxx"></ref>
        </entry>
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

##### 特殊值
XML实体字符：
* `"`：`&quot;`
* `&`：`&amp;`
* `'`：`&apos;`
* `<`：`&lt;`
* `>`：`&gt;`

```XML
<!-- 小于号在XML文档中用来定义标签的开始，不能随便使用 -->
<!-- 解决方案一：使用XML实体来代替 -->
<property name="expression" value="a &lt; b"/>

<property name="expression">
    <!-- 解决方案二：使用CDATA节 -->
    <!-- CDATA中的C代表Character，是文本、字符的含义，CDATA就表示纯文本数据 -->
    <!-- XML解析器看到CDATA节就知道这里是纯文本，就不会当作XML标签或属性来解析 -->
    <!-- 所以CDATA节中写什么符号都随意 -->
    <value><![CDATA[a < b]]></value>
</property>
```

### FactoryBean
FactoryBean是Spring提供的一种整合第三方框架的常用机制。和普通的bean不同，配置一个FactoryBean类型的bean，在获取bean的时候得到的并不是class属性中配置的这个类的对象，而是getObject()方法的返回值。通过这种机制，Spring可以帮我们把复杂组件创建的详细过程和繁琐细节都屏蔽起来，只把最简洁的使用界面展示给我们。

创建示例类：
```Java
package com.atguigu.spring6.bean;
public class UserFactoryBean implements FactoryBean<User> {
    @Override
    public User getObject() throws Exception {
        return new User();
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }
}
```

配置bean：
```XML
<bean id="user" class="com.atguigu.spring6.bean.UserFactoryBean"></bean>
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
<!-- 引入 context 名称空间 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
</beans>

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

### 基于xml自动装配
自动装配：根据指定的策略，在IOC容器中匹配某一个bean，自动为指定的bean中所依赖的类类型或接口类型属性赋值。

使用bean标签的autowire属性设置自动装配效果：
```XML
<!-- 自动装配方式：byType，根据类型匹配IOC容器中的某个兼容类型的bean，为属性自动赋值 -->
<bean id="userController" class="com.atguigu.spring6.autowire.controller.UserController" autowire="byType"></bean>

<bean id="userService" class="com.atguigu.spring6.autowire.service.impl.UserServiceImpl" autowire="byType"></bean>

<bean id="userDao" class="com.atguigu.spring6.autowire.dao.impl.UserDaoImpl"></bean>

<!-- 此处为分割线 -->

<!-- 自动装配方式：byName，将自动装配的属性的属性名，作为bean的id在IOC容器中匹配相对应的bean进行赋值 -->
<bean id="userController" class="com.atguigu.spring6.autowire.controller.UserController" autowire="byName"></bean>

<bean id="userService" class="com.atguigu.spring6.autowire.service.impl.UserServiceImpl" autowire="byName"></bean>

<bean id="userDao" class="com.atguigu.spring6.autowire.dao.impl.UserDaoImpl"></bean>
```



## 注解 的 配置方式
从 Java 5 开始，Java 增加了对注解（Annotation）的支持，它是代码中的一种特殊标记，可以在编译、类加载和运行时被读取，执行相应的处理。开发人员可以通过注解在不改变原有代码和逻辑的情况下，在源代码中嵌入补充信息。Spring 从 2.5 版本开始提供了对注解技术的全面支持，我们可以使用注解来实现自动装配，简化 Spring 的 XML 配置。

Spring 通过注解实现自动装配的步骤如下：
1. 引入依赖
2. 开启组件扫描
3. 使用注解定义 Bean
4. 依赖注入

### 开启组件扫描
Spring 默认不使用注解装配 Bean，因此我们需要在 Spring 的 XML 配置中，通过 `<context:component-scan>` 元素开启 Spring Beans的自动扫描功能。开启此功能后，Spring 会自动从扫描指定的包（base-package 属性设置）及其子包下的所有类，如果类上使用了 @Component 注解，就将该类装配到容器中。

需要在xml配置文件中做如下配置：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 最基本的扫描方式：扫描哪个包下所有的类，指定基包的名字（可以使用逗号分隔多个基包）-->
    <context:component-scan base-package="com.itheima"/>
    
    <!-- 指定要排除的组件 -->
    <context:component-scan base-package="com.atguigu.spring6">
        <!-- context:exclude-filter标签：指定排除规则 -->
        <!-- 
            type：设置排除或包含的依据
            type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
            type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
        -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:exclude-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>
    </context:component-scan>

    <!-- 仅扫描指定组件 -->
    <context:component-scan base-package="com.atguigu" use-default-filters="false">
        <!-- context:include-filter标签：指定在原有扫描规则的基础上追加的规则 -->
        <!-- use-default-filters属性：取值false表示关闭默认扫描规则 -->
        <!-- 此时必须设置use-default-filters="false"，因为默认规则即扫描指定包下所有类 -->
        <!-- 
            type：设置排除或包含的依据
            type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
            type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
        -->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:include-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>
    </context:component-scan>
</beans>
```

### 常用注解

#### @Component
Spring 提供了以下多个注解，这些注解可以直接标注在 Java 类上，将它们定义成 Spring Bean。

默认是类名首字母小写做为名字，也可以使用value属性来指定*该类的名字*（相当于配置文件中`bean id="该类的名字" class="当前注解的类的全类名"/>`）。

| 创建对象的注解 | 说明           |
| -------------- | -------------- |
| @Component     | 放在普通的类上 |
| @Controller    | 放在控制器     |
| @Service       | 放在业务对象   |
| @Repository    | 放在持久层     |

以上四个注解（@Component是另三个注解的父注解）的功能一样，只是语义上区别。

#### @Autowired
Autowired注解放在属性上、构造方法上、构造方法的参数上、setter方法上，表示将*相应的值*注入该属性（从容器中寻找合适的被依赖资源）。*相应的值*：
* 按类型匹配的方式从容器中去查找对应的值注入
* 如果有多个匹配的类型，按名字匹配的方式注入
* 如果找不到匹配的名字，就会抛出异常

@Autowired的属性required：默认为true，表示这个属性是必需的；如果为false，则匹配失败时，不抛出异常，将属性值为null。

当带参数的构造方法只有一个，@Autowired注解可以省略。

#### @Qualifier
Autowired是根据类型自动装配的，加上@Qualifier则可以根据byName的方式自动装配：
* 位置: 放在成员变量或成员方法上
* 作用: 按名字匹配的方式注入
* 属性: value指定要注入的名字

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

    @Value("file:/path/to/logo.txt")
    private Resource resource;
}
```

#### @PropertySource
Spring容器还提供了一个更简单的@PropertySource来自动读取配置文件。

先使用@PropertySource读取配置文件，然后通过@Value以`${key:defaultValue}`的形式注入，可以极大地简化读取配置的麻烦。

#### @Scope
用在类上，用来指定当前对象在容器中是单例对象(singleton)还是多例对象(prototype)。

#### @Lazy
用在类上，用来指定当前对象是否延迟加载，在默认的情况(false)下，容器加载对象就加载了。如果为true表示延迟加载。

#### @PostConstruct
用在方法上，指定这是一个初始化的方法。Spring只根据Annotation查找无参数方法，对方法名不作要求。

#### @PreDestroy
用在方法上，指定这是对象销毁前执行的方法，只适用于单例对象。Spring只根据Annotation查找无参数方法，对方法名不作要求。

#### @ComponentScan
自动搜索当前类所在的包以及子包，把所有标注为@Component的Bean自动创建出来，并根据@Autowired进行装配。

#### @Configuration
从Spring3.0，@Configuration用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

#### @Bean
可用带@Bean标注的方法创建Bean（作为方法的返回值），Spring对标记为@Bean的方法只调用一次，因此返回的Bean仍然是单例。可以用@Bean("*名字*")指定别名，也可以用@Bean+@Qualifier("*名字*")指定别名。

#### @Primary
存在多个同类型的Bean时，把其中某个Bean指定为@Primary，在注入时，如果没有指出Bean的名字，Spring会注入标记有@Primary的Bean。

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



## Java配置类 的 配置方式
Java配置类的配置方式，可以不再使用spring配置文件了，写一个配置类来代替配置文件。
```Java
// 配置类
package com.atguigu.spring6.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
//@ComponentScan({"com.atguigu.spring6.controller", "com.atguigu.spring6.service","com.atguigu.spring6.dao"})
@ComponentScan("com.atguigu.spring6")
public class Spring6Config {
}

// 测试类
@Test
public void testAllAnnotation(){
    ApplicationContext context = new AnnotationConfigApplicationContext(Spring6Config.class);
    UserController userController = context.getBean("userController", UserController.class);
    userController.out();
    logger.info("执行成功");
}
```




## Spring XML中常用的命名空间
* p命名空间：`xmlns:p="http://www.springframework.org/schema/p"`
* c命名空间：`xmlns:c="http://www.springframework.org/schema/c"`
* util命名空间：`xmlns:util="http://www.springframework.org/schema/util"`
* context命名空间：`xmlns:context="http://www.springframework.org/schema/context"`