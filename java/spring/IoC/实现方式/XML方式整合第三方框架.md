# XML方式整合第三方框架

xml整合第三方框架有两种整合方案：
1. **不需要自定义名空间，不需要使用Spring的配置文件配置第三方框架本身内容**，例如：MyBatis;
2. **需要引入第三方框架命名空间，需要使用Spring的配置文件配置第三方框架本身内容**，例如：Dubbo。

## MyBatis
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