# XML文件配置Bean



## bean标签
bean标签用于描述被IOC管理的对象，它有如下几个最常用的属性：
|属性|说明|
|---|---|
|id|容器中唯一的标识（**beanName**）。默认值为当前Bean实例的全限定名|
|name|通过name设置Bean的**别名**，通过别名也能直接获取到Bean实例。可以有多个别名，使用*逗号*、*空格*、*分号*隔开都可以|
|class|指定全限定名（是实现类，不是接口）|
|scope| 指定bean在容器中作用范围<br />singleton：默认值，表示这个是单例对象，整个容器中只会创建一个对象；容器一创建就创建这个对象，并存储到容器内部的单例池中，只要容器不销毁就一直存在；每次getBean时都是从单例池中获取相同的实例<br />prototype：这是多例对象，每次获取对象就创建一个新的对象，使用完毕会被GC回收；单例池中不存在对象的实例，但是对象的信息已经被存储到beanDefinitionMap中<br />request：用于请求域<br />session：用于会话域<br />application: 用于上下文域<br />globalsession：用于全局的会话，用在分布式开发中|
|lazy-init|是否使用延迟加载，默认值为false；延迟加载只用于单例对象|
|init-method|Bean实例化后自动执行的初始化方法，method指定方法名|
|destroy-method|Bean实例销毁前的方法，method指定方法名；销毁的方法只在单例模式下起作用|
|autowire|设置自动注入模式，常用的有按照类型byType，按照名字byName|
|factory-bean|指定工厂Bean|
|factory-method|搭配factory-bean属性（实例工厂）或class属性（静态工厂），指定工厂方法|

bean标签的内容（子标签）主要适用于描述该对象的依赖对象。



## bean的创建和注入
创建对象方式：
* 构造方法
  * 无参（默认，setter注入）
  * 有参（构造器注入）
* 工厂方法
  * 静态工厂方法
  * 实例工厂方法
* FactoryBean

### 静态工厂方法
静态工厂方法实例化Bean，其实就是定义一个工厂类，提供一个静态方法用于生产Bean实例，在将该工厂类及其 静态方法配置给Spring即可。
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
<!-- 使用静态工厂创建
    class：静态工厂类
    factory-method：工厂中的获取对象的静态方法
    id：对象的id
-->
<bean class="com.itheima.utils.StaticBeanFactory" factory-method="getBean" id="customerService"/>
```

### 实例工厂方法
实例工厂方法，也就是非静态工厂方法产生Bean实例，与静态工厂方式比较，该方式需要先有工厂对象，在用工厂 对象去调用非静态方法，所以在进行配置时，要先配置工厂Bean，再配置目标Bean。单例池中既有工厂Bean实例，也有目标Bean实例。
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
<!-- 使用实例工厂来创建
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

### 依赖注入（通过构造器或Setter方法）
Bean为：
```Java
public class Customer {
    private int id;
    private String name;
    private boolean male;
    private Date birthday;
    // 省略无参构造方法、有参构造方法、getter和setter
}
```

#### 构造器注入
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
    index：指定形参的位置，从0开始
    name：指定形参的名字
    type：指定形参的类型
    value：注入简单类型（8种基本类型和String类型）
    ref：注入引用类型，指定容器中另一个bean的id属性
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

##### c命名空间
```xml
<!-- 导入约束：xmlns:c="http://www.springframework.org/schema/c" -->

<!--C(构造：Constructor)命名空间-->
<bean id="user" class="com.kuang.pojo.User" c:name="狂神" c:age="18"/>
 ```

#### setter注入
|property标签的属性|描述|
|---|---|
|name|属性名|
|value|注入简单类型的值|
|ref|注入引用类型的值|


```xml
<!-- 2.使用set方法注入 -->
<bean class="com.itheima.entity.Customer" id="customer">
    <!-- 属性：name为id，value为200 -->
    <property name="id" value="200"/>

    <property name="name" value="猪八戒"/>
    <property name="male" value="true"/>
    <property name="birthday" ref="birthday"/>
</bean>

<!-- 引用类型：现在的时间 -->
<bean class="java.util.Date" id="birthday"/>
```

##### p命名空间
```xml
<!-- 导入约束：xmlns:p="http://www.springframework.org/schema/p" -->
 
<!--P(属性：properties)命名空间, 本质上还是set注入-->
<bean id="user" class="com.kuang.pojo.User" p:name="狂神" p:age="18" p:birthday-ref="birthday"/>

<!-- 引用类型：现在的时间 -->
<bean class="java.util.Date" id="birthday"/>
```

#### 依赖注入示例
依赖注入的数据类型有如下三种：
* 普通数据类型，通过value属性指定。
* 引用数据类型，通过ref属性指定。
* 集合数据类型。

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
    private List<Address> objList;
    // 省略getter和setter
}
```

```xml
<!-- 普通数据类型 -->
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
</bean>

<!-- 引用数据类型 -->
<!-- Bean -->
<bean id="addr1" class="com.kuang.pojo.Address">
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
        <bean id="addr2" class="com.kuang.pojo.Address">
            <property name="address" value="重庆"/>
        </bean>
    </property>
</bean>

<!-- 集合数据类型 -->
<bean id="student" class="com.kuang.pojo.Student">
    <!-- List -->
    <property name="hobbys">
        <list>
            <value>听歌</value>
            <value>看电影</value>
            <value>爬山</value>
        </list>
    </property>
    <!-- List(对象) -->
    <property name="objList">
        <list>
            <bean id="addr3" class="com.kuang.pojo.Address">
                <property name="address" value="重庆"/>
            </bean>
            <bean class="com.kuang.pojo.Address">
                <property name="address" value="重庆"/>
            </bean>
            <ref bean="addr3"></ref>
        </list>
    </property>
    <!-- 数组 -->
    <property name="books">
        <array>
            <value>西游记</value>
            <value>红楼梦</value>
            <value>水浒传</value>
        </array>
    </property>
    <!-- Set -->
    <property name="games">
        <set>
            <value>LOL</value>
            <value>BOB</value>
            <value>COC</value>
        </set>
    </property>
    <!-- Properties -->
    <property name="info">
        <props>
            <prop key="学号">20190604</prop>
            <prop key="性别">男</prop>
            <prop key="姓名">小明</prop>
        </props>
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
            <entry key="工商" value-ref="yyy"/>
        </map>
    </property>
    <!-- null -->
    <property name="wife">
        <null/>
    </property>
</bean>
```

#### 特殊值
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

#### 自动装配
如果被注入的属性类型是Bean引用的话，那么可以在bean标签中使用autowire属性去配置自动注入方式，属性值有两个：
* byName：通过属性名自动装配，即去匹配 setXxx 与 id="xxx"(name="xxx")是否一致;
* byType：通过Bean的类型从容器中匹配，匹配出多个相同Bean类型时报错。

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

### FactoryBean
FactoryBean是Spring提供的一种整合第三方框架的常用机制。和普通的bean不同，配置一个FactoryBean类型的bean，在获取bean的时候得到的并不是class属性中配置的这个类的对象，而是getObject()方法的返回值。

Spring容器创建时，FactoryBean被实例化了，并存储到了单例池singletonObjects中，但是getObject()方法尚未被执行，Bean也没被实例化，当首次用到Bean时，才调用getObject()。此方式产生的Bean实例不会存储到单例池singletonObjects中，会存储到factoryBeanObjectCache缓存池中，并且后期每次使用到Bean都从该缓存池中返回的是同一个Bean实例。

FactoryBean接口定义：
```Java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = “factoryBeanObjectType”;
    T getObject() throwsException; //获得实例对象方法
    Class<?> getObjectType(); //获得实例对象类型方法
    default boolean isSingleton() {
        return true;
    }
}
```

创建示例类：
```Java
package com.atguigu.spring6.bean;
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    @Override
    public User getObject() throws Exception {
        return new UserDaoImpl();
    }

    @Override
    public Class<?> getObjectType() {
        return UserDao.class;
    }
}
```

配置bean：
```XML
<bean id="userDao" class="com.atguigu.spring6.bean.UserDaoFactoryBean"></bean>
```



## 其他配置标签
Spring 的 xml 标签大体上分为两类，一种是默认标签，一种是自定义标签：
* 默认标签：不用额外导入其他命名空间约束的标签，例如`<bean>`标签
* 自定义标签：需要额外引入其他命名空间约束，并通过前缀引用的标签，例如`<context:property- placeholder/>`标签

### 自定义标签
Spring的自定义标签需要引入外部的命名空间，并为外部的命名空间指定前缀，使用`<前缀:标签>`形式的标签，称之为自定义标签，自定义标签的解析流程也是Spring xml扩展点方式之一。
```XML
<!--自定义标签-->

<!-- 加载properties文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>
<!-- 组件扫描 -->
<context:component-scan base-package="com.itheima"/>
<mvc:annotation-driven/>
<dubbo:application name="application"/>
```

### 默认标签
Spring的默认标签用到的是Spring的默认命名空间：
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

该命名空间约束下的默认标签如下：
* `<beans>`：一般作为 xml 配置根标签，其他标签都是该标签的子标签
* `<bean>`：Bean的配置标签，上面已经详解了，此处不再阐述
* `<import>`：外部资源导入标签
* `<alias>`：指定Bean的别名标签，使用较少

#### beans标签
`<beans>`标签，除了经常用的做为根标签外，还可以嵌套在根标签内，使用profile属性切换开发环境：
```XML
<!-- 配置测试环境下，需要加载的Bean实例 -->
<beans profile="test">
  ...
</beans>

<!-- 配置开发环境下，需要加载的Bean实例 -->
<beans profile="dev">
  ...
</beans>
```

#### alias标签
`<alias>`标签是为某个Bean添加别名，与在`<bean>`标签上使用name属性添加别名的方式一样。为 UserServiceImpl指定四个别名：
```xml
<!--配置UserService-->
<bean id="userService" name="aaa,bbb" class="com.itheima.service.impl.UserServiceImpl">
    <property name="userDao" ref="userDao"/>
</bean>

<!--设置别名：在获取Bean的时候可以使用别名获取-->
<alias name="userService" alias="xxx"/>
<alias name="userService" alias="yyy"/>
```
在beanFactory中维护着一个名为aliasMap的Map<String,String>集合，存储别名和beanName之间的映射关系：
![IoC+20231109185943](https://raw.githubusercontent.com/loli0con/picgo/master/images/IoC%2B20231109185943.png%2B2023-11-09-18-59-44)

#### import标签
`<import>`标签，用于导入其他配置文件，项目变大后，就会导致一个配置文件内容过多，可以将一个配置文件根据业务某块进行拆分，拆分后，最终通过`<import>`标签导入到一个主配置文件中，项目加载主配置文件会连同`<import>`导入的文件一并加载：
1. 配置文件会先合并后解析，也就是说，无论是命名空间还是配置的内容，都会合并处理（合并前的报错请忽略）。
2. 多个Spring配置文件最终会合并到一起，形成一个配置文件，因此这些配置中的bean都是可以互相引用的。

```xml
<import resource="路径/beans.xml"/>
```



## 配置非自定义Bean
在实际开发中有些 功能类并不是我们自己定义的，而是使用的第三方jar包中的，那么，这些Bean要想让Spring进行管理，也需要对其进行配置。配置非自定义的Bean需要考虑如下两个问题：
1. 被配置的Bean的实例化方式：无参构造、有参构造、静态工厂方式、实例工厂方式和FactoryBean
2. 被配置的Bean是否需要注入必要属性