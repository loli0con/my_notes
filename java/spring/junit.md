# Spring + JUnit

## @RunWith（JUnit4）
JUnit4 提供了一个注解@RunWith，可以替换掉它自己本身的运行器。  
Spring提供了SpringJUnit4ClassRunner运行器。

## @ExtendWith（Junit5）
JUnit5 提供了一个注解@ExtendWith，可以替换掉它自己本身的运行器。  
Spring提供了SpringExtension运行器。

## @ContextConfiguration
使用Spring提供的注解@ContextConfiguration，可以读取配置文件（或注解）来创建Spring容器。

## 示例

### @RunWith
```Java
@RunWith(SpringJUnit4ClassRunner.class) // JUnit4
@ContextConfiguration(locations="classpath:applicationContext.xml")  // 指定配置文件（和⬇️一行代码二选一）
@ContextConfiguration(classes = SpringConfig.class)// 指定配置类（和⬆️一行代码二选一）
public class T1 {
    /* 测试类 */
}
```

### @ExtendWith
```Java
@ExtendWith(SpringExtension.class) // JUnit5
@ContextConfiguration(locations="classpath:applicationContext.xml")  // 指定配置文件（和⬇️一行代码二选一）
@ContextConfiguration(classes = SpringConfig.class)// 指定配置类（和⬆️一行代码二选一）
public class T2 {
    /* 测试类 */
}
```

### @SpringJUnitConfig
```Java
@SpringJUnitConfig(locations = "classpath:applicationContext.xml") // JUnit5
// @SpringJUnitConfig(classes = SpringConfig.class)
public class T3 {
    /* 测试类 */
}
```


