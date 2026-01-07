# XML方式整合第三方框架

xml整合第三方框架有两种整合方案：
1. **不需要自定义名空间，不需要使用Spring的配置文件配置第三方框架本身内容**，例如：[MyBatis](#mybatis不需要自定义名空间);
2. **需要引入第三方框架命名空间，需要使用Spring的配置文件配置第三方框架本身内容**，例如：[Dubbo](#dubbo需要引入第三方框架命名空间)。

## MyBatis（不需要自定义名空间）
MyBatis提供了mybatis-spring.jar专门用于两大框架的整合。

## 步骤
Spring整合MyBatis的步骤如下：
1. 导入MyBatis整合Spring的相关坐标;
2. 编写Mapper和Mapper.xml;
3. 配置SqlSessionFactoryBean和MapperScannerConfigurer;
4. 编写测试代码。

### 配置SqlSessionFactoryBean和MapperScannerConfigurer
```XML
<!--配置数据源-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis"></property> <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
<!--配置SqlSessionFactoryBean-->
<bean class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!--配置Mapper包扫描-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.itheima.dao"></property>
</bean>
```

### 编写Mapper
```Java
public interface UserMapper {
    List<User> findAll();
}
```

### 编写Mapper.xml
```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.UserMapper">
    <select id="findAll" resultType="com.itheima.pojo.User">
    select * from tb_user
    </select>
</mapper>
```

### 编写测试代码
```Java
ClassPathxmlApplicationContext applicationContext = new ClassPathxmlApplicationContext("applicationContext.xml");
UserMapper userMapper = applicationContext.getBean(UserMapper.class);
List<User> all = userMapper.findAll();
System.out.println(all);
```

## 原理
整合包里提供了一个SqlSessionFactoryBean和一个扫描Mapper的配置对象，SqlSessionFactoryBean一旦被实例化，就开始扫描Mapper并通过动态代理产生Mapper的实现类存储到Spring容器中。相关的有如下四个类：
1. SqlSessionFactoryBean: 需要进行配置，用于提供SqlSessionFactory;
2. MapperScannerConfigurer: 需要进行配置，用于扫描指定mapper注册BeanDefinition; 
3. MapperFactoryBean: Mapper的FactoryBean，获得指定Mapper时调用getObject方法;
4. ClassPathMapperScanner: `definition.setAutowireMode(2);`修改了自动注入状态，所以MapperFactoryBean中的setSqlSessionFactory会自动注入进去。

### SqlSessionFactoryBean
配置SqlSessionFactoryBean作用是向容器中提供SqlSessionFactory，SqlSessionFactoryBean实现了FactoryBean和InitializingBean两个接口，所以会自动执行getObject()和afterPropertiesSet()方法：
```Java
SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean{
    public void afterPropertiesSet() throws Exception {
        //创建SqlSessionFactory对象
        this.sqlSessionFactory = this.buildSqlSessionFactory();
    }
    public SqlSessionFactory getObject() throws Exception {
        return this.sqlSessionFactory;
    }
}
```

### MapperScannerConfigurer
配置MapperScannerConfigurer作用是扫描Mapper，向容器中注册Mapper对应的MapperFactoryBean，MapperScannerConfigurer实现了BeanDefinitionRegistryPostProcessor和InitializingBean两个接口，会在postProcessBeanDefinitionRegistry方法中向容器中注册MapperFactoryBean：
```Java
// 以下只有关键代码，非关键代码会被省略！！！
// 以下只有关键代码，非关键代码会被省略！！！
// 以下只有关键代码，非关键代码会被省略！！！

class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean{ 
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
        ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
        scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ",; \t\n"));
    } 
}

class ClassPathMapperScanner extends ClassPathBeanDefinitionScanner {
    public Set<BeanDefinitionHolder> doScan(String... basePackages) {
        Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
        if (beanDefinitions.isEmpty()) {
        } else {
            this.processBeanDefinitions(beanDefinitions);
        }
    }
    private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
        // 遍历beanDefinitions：
        // 设置Mapper的beanClass是org.mybatis.spring.mapper.MapperFactoryBean
        // 因为扫描到的是Interface（示例中为com.itheima.mapper.UserMapper），无法创建对象
        definition.setBeanClass(this.mapperFactoryBeanClass);

        // 设置MapperBeanFactory进行自动注入()
        // autowireMode取值: 1是根据名称自动装配，2是根据类型自动装配
        // 该definition产生的Bean为MapperFactoryBean，其getObject方法见下
        definition.setAutowireMode(2);
        
    }
}

class ClassPathBeanDefinitionScanner{
    public int scan(String... basePackages) {
        this.doScan(basePackages);
    }
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        // definitionHolder包装了beanDefinition和beanName（它的两个属性）
        // 将扫描到的类注册到beanDefinitionMap中
        // 此时beanClass是当前类全限定名（示例中为com.itheima.mapper.UserMapper）
        this.registerBeanDefinition(definitionHolder, this.registry);

        // 完成所有的注册后，返回集合
        return beanDefinitions;
    }
}

public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {
    public MapperFactoryBean(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionTemplate = this.createSqlSessionTemplate(sqlSessionFactory);
    }
    public T getObject() throws Exception {
        // this.getSqlSession()，需要用到SqlSession，SqlSession通过SqlSessionFactory产生
        // 通过definition.setAutowireMode(2)，让该bean对所需的属性按类型进行自动注入
        // 即完成setSqlSessionFactory的注入
        return this.getSqlSession().getMapper(this.mapperInterface);
    }
}
```

## Dubbo（需要引入第三方框架命名空间）
此处以Spring的**context**命名空间去进行讲解，该方式也是命名空间扩展方式。

### 用法
加载外部properties文件，将键值对存储在Spring容器中：
```properties
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=root
```

```XML
引入context命名空间，再使用context命名空间的标签：
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
       <!-- 上面最后两行：引入context命名空间 + 对应的XSD文件 -->
    
    <!-- 下面使用context命名空间的标签 -->
    <context:property-placeholder location="classpath:jdbc.properties" />

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>
<beans>
```

### 原理分析
从源头ClassPathXmlApplicationContext入手，找到处理每一个XML元素结点的代码处：
![XML方式整合第三方框架+20231114072743](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114072743.png%2B2023-11-14-07-27-45)

如果是默认的命名空间的XML元素，则调用`this.parseDefaultElement`方法：
![XML方式整合第三方框架+20231114073037](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114073037.png%2B2023-11-14-07-30-39)

如果不是默认的命名空间的XML元素，则调用`delegate.parseCustomElement`方法。通过`NamespaceHandlerResolver`的`resolver`方法，传入该命名空间的Uri，获取该命名空间的`NamespaceHandler`：
![XML方式整合第三方框架+20231114073423](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114073423.png%2B2023-11-14-07-34-25)

`NamespaceHandlerResolver`的`resolver`方法根据XML元素所处的命名空间Uri，从`handlerMappings`中获取对应的命名空间处理器`NamespaceHandler`，该对象后续将调用`parse`方法解析自定义的XML元素：
![XML方式整合第三方框架+20231114073748](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114073748.png%2B2023-11-14-07-37-51)

`handlerMappings`作为`DefualtNamespeceHandlerResolver`的成员变量，在构造方法内根据`handlerMappingsLocation:"META-INF/spring.handlers"`进行初始化：
![XML方式整合第三方框架+20231114074256](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114074256.png%2B2023-11-14-07-42-58)

`META-INF/spring.handlers`的内容如下图所示：
![XML方式整合第三方框架+20231114074624](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114074624.png%2B2023-11-14-07-46-27)

`NamespaceHandlerResolver`的`resolver`方法获取对应的命名空间处理器`NamespaceHandler`后，会确保调用过该`NamespaceHandler`的`init`方法完成初始化后才返回：
![XML方式整合第三方框架+20231114075055](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114075055.png%2B2023-11-14-07-50-57)

`NamespaceHandler`的`init`方法通常用于注册命名空间下的标签的解析器`Parser`到其父类 `NamespaceHandlerSupport`的`Map<String, BeanDefinitionParser> parsers`中，一个标签对应一个`Parser`：
![XML方式整合第三方框架+20231114085554](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114085554.png%2B2023-11-14-08-55-57)

回到上层的`delegate.parseCustomElement`方法，从`NamespaceHandlerResolver`中获取到对应`handler`后，调用`parser`方法，解析自定义的XML元素：
![XML方式整合第三方框架+20231114084814](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114084814.png%2B2023-11-14-08-48-18)

### 总结原理
1. **命名空间的名称** 与 **命名空间的处理器映射关系** 会以键值对形式存在到一个叫`spring.handlers`文件里，该文件存储在类加载路径的`META-INF`目录下，Spring会自动加载
2. `NamespaceHandler`命名空间处理器：如果命名空间只有一个标签，那么直接在`parse`方法中进行解析即可，一般解析结 果就是注册该标签对应的`BeanDefinition`。如果命名空间里有多个标签，那么可以在`init`方法中为每个标签都注册一个`BeanDefinitionParser`，在执行`NamespaceHandler`的`parse`方法时在分流给不同的 `BeanDefinitionParser`进行解析(重写`doParse`方法即可)。

### 虚拟XSD
![XML方式整合第三方框架+20231114232920](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114232920.png%2B2023-11-14-23-29-23)

![XML方式整合第三方框架+20231114233356](https://raw.githubusercontent.com/loli0con/picgo/master/images/XML%E6%96%B9%E5%BC%8F%E6%95%B4%E5%90%88%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6%2B20231114233356.png%2B2023-11-14-23-33-59)