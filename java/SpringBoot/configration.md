# 配置文件
SpringBoot提供了很多的预设配置，我们也可以按照自己的需要进行手动配置，修改SpringBoot自动配置的默认值。

## 配置格式
SpringBoot使用一个全局的配置文件，**配置文件名称是固定的**，支持两种格式：
* application.properties
* application.yml（**官方推荐**）

### yaml
YAML 是专门用来写配置文件的语言，非常简洁和强大。

SpringBoot强烈推荐yaml配置，并且基于native YAML提供更强大的特性支持。

请按需阅读：
1. [YAML 入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)
2. [YAML 语言教程](https://www.ruanyifeng.com/blog/2016/07/yaml.html)

## 配置注入
可以使用配置文件中的值注入到实体类

### 配置提示
```xml
<!-- 导入配置文件处理器，配置文件进行绑定就会有提示 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring‐boot‐configuration‐processor</artifactId>
  <optional>true</optional>
</dependency>
```

### yaml-demo

#### yaml配置文件
```yml
person:
  full-name: qinjiang${random.uuid}
  age: ${random.int}
  happy: false
  birth: 2000/01/01
  maps: {k1: v1,k2: v2}
  lists:
   - code
   - girl
   - music
  dog:
    name: ${person.full-name:other}_旺财
    age: 1
```

#### 实体类
```java
/*
@ConfigurationProperties作用：
将配置文件中配置的每一个属性的值，映射到这个组件中；
告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
参数 prefix = “person” : 将配置文件中的person下面的所有属性一一对应
*/
@Component //注册bean
@ConfigurationProperties(prefix = "person")
public class Person {
    private String fullName;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}

@Component  //注册bean到容器中
@Validated // 开启数据校验
public class Dog {
    private String name;

    @Size(0, 20) // JSR303数据校验
    private Integer age;
    
    //有参无参构造、get、set方法、toString()方法  
}
```

#### 测试类
```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    Person person; //将person自动注入进来

    @Test
    public void contextLoads() {
        System.out.println(person); //打印person信息
    }

}
```

### 常用注解
* @PropertySource：加载指定的配置文件（完成加载后，根据配置文件的类型，使用下面2个不同的注解进行值注入）
  * value属性：指定配置文件(数组类型，可加载多个)
* yaml - @configurationProperties：默认从全局配置文件中获取值
  * prefix属性：指定配置文件中绑定到被注入实体类的配置内容
* 读取properties - @Value：给被标注的属性注入值，值可以有多种来源：字面量/Spring表达式/从配置文件中取值
* @ImportResource：读取外部配置文件

### yaml-占位符
yaml配置文件还可以使用占位符。

#### 随机值占位符
使用占位符生成随机数，示例：
* ${random.value}
* ${random.int}
* ${random.long}
* ${random.int(10)}
* ${random.int\[1024,65536]}

#### 属性配置占位符
使用占位符可以在配置文件中引用前面配置过的属性，如果没有也可以使用指定默认值：
* ${person.full-name:other}：获取person对象的full-name属性的值，如果找不到属性则使用other值(默认值)。


### yaml-松散绑定
Spring Boot使用一些宽松的规则用于绑定Environment属性到@ConfigurationProperties beans，所以Environment属性名和bean属性名不需要精确匹配。

下面的属性名都能用于示例中的@ConfigurationProperties类：
| 属性             | 说明                                      |
| ---------------- | ----------------------------------------- |
| person.fullName  | 标准驼峰规则                              |
| person.full-name | 虚线表示，推荐用于.properties和.yml文件中 |
| PERSON_FULL_NAME | 大写形式，使用系统环境变量时推荐          |

### yaml-JSR303数据校验
简述 JSR303/JSR-349，hibernate validation，spring validation 之间的关系：
* JSR303 是一项标准,JSR-349 是其的升级版本，添加了一些新特性，他们规定一些校验规范即校验注解，如 @Null，@NotNull，@Pattern，他们位于 javax.validation.constraints 包下，只提供规范不提供实现
* 而 hibernate validation 是对这个规范的实践（不要将 hibernate 和数据库 orm 框架联系在一起），他提供了相应的实现，并增加了一些其他校验注解，如 @Email，@Length，@Range 等等，他们位于 org.hibernate.validator.constraints 包下。
* 而万能的 spring 为了给开发者提供便捷，对 hibernate validation 进行了二次封装，显示校验 validated bean 时，你可以使用 spring validation 或者 hibernate validation，而 spring validation 另一个特性，便是其在 springmvc 模块中添加了自动校验，并将校验信息封装进了特定的类中。
 
这无疑便捷了我们的 web 开发。
除此以外，我们还可以自定义一些数据校验规则，满足特定的需求。

使用时，需要给目标对象（实体类、方法参数等）加上@Validated注解，告诉Spring对此进行校验。

#### Bean Validation 中内置的 constraint
| Constraint                  | 详细信息                                                 |
| --------------------------- | -------------------------------------------------------- |
| @Null                       | 被注释的元素必须为 null                                  |
| @NotNull                    | 被注释的元素必须不为 null                                |
| @AssertTrue                 | 被注释的元素必须为 true                                  |
| @AssertFalse                | 被注释的元素必须为 false                                 |
| @Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size(max, min)             | 被注释的元素的大小必须在指定的范围内                     |
| @Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Past                       | 被注释的元素必须是一个过去的日期                         |
| @Future                     | 被注释的元素必须是一个将来的日期                         |
| @Pattern(value)             | 被注释的元素必须符合指定的正则表达式                     |

#### Hibernate Validator 附加的 constraint
| Constraint | 详细信息                               |
| ---------- | -------------------------------------- |
| @Email     | 被注释的元素必须是电子邮箱地址         |
| @Length    | 被注释的字符串的大小必须在指定的范围内 |
| @NotEmpty  | 被注释的字符串的必须非空               |
| @Range     | 被注释的元素必须在合适的范围内         |


### 配置复用
我们可以将那些都需要用到的属性抽离出来，放到一个公共的属性类中，实现代码复用

#### 公共属性类
```java
@ConfigurationProperties(prefix = "person")
public class PersonProperties {
    // 省略代码
}
```

#### 测试类
```java
@Controller
@EnableConfigurationProperties(PersonProperties.class) // 指定某类生成Bean
public class PropController4 {
    @Autowired
    private PersonProperties personProperties;

    // 省略代码
}
```


## 配置位置
springboot启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件：
1. 项目路径下的config文件夹配置文件（file:./config/）
2. 项目路径下配置文件（file:./）
3. 资源路径下的config文件夹配置文件（classpath:/config/）
4. 资源路径下配置文件（classpath:/）

优先级由高到底，高优先级的配置会覆盖低优先级的配置。

在同一级目录下优先级为：properties > yml > yaml，相同的属性取优先级高的文件中的值，不同的属性取并集（项目中一般只用一种文件）。

## profile - 多环境
profile是Spring对不同环境提供不同配置的功能支持，可以通过激活不同的环境版本，实现快速切换环境。

### 方案1 - 多profile文件形式
我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml，用来指定多个环境版本，例如：
* application-test.properties 代表测试环境配置
* application-dev.properties 代表开发环境配置

但是Springboot并不会直接启动这些配置文件，它默认使用application.properties主配置文件，我们需要通过一个配置（在主配置文件中）来选择需要激活的环境。

#### 配置文件参数激活环境
properties格式：
```properties
# 比如在配置文件中指定使用dev环境，我们可以通过设置不同的端口号进行测试；
# 我们启动SpringBoot，就可以看到已经切换到dev下的配置了；
spring.profiles.active=dev
```

yml格式：
```yml
spring:
  profiles:
    active: dev # 指定想要激活环境的key
```

### 方案2 - yaml的多文档块
使用yml去实现该功能（使用“---”分割不同的配置），不需要创建多个配置文件就可以达到和properties配置文件中一样效果：
```yml
server:
  port: 8081
#选择要激活那个环境块
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev #配置环境的名称

---
server:
  port: 8084
spring:
  profiles: prod  #配置环境的名称
```

### 传入参数激活不同的环境
除了上面提到的[配置文件参数激活环境](#配置文件参数激活环境)外，还可以通过如下方式激活环境。
#### jvm参数
```sh
java -jar -Dspring.profiles.active=环境key jar包名称  # 切记-D参数一定要写在jar包前面
# -D<名称>=<值> 用于 设置系统属性（临时系统属性，供运行的java程序读取）
```
#### 命令行参数
```sh
java -jar jar包名称 --spring.profiles.active=环境key  # 切记--参数一定要写在jar包后面
# 这里设置的是传递给java程序的命令行参数
```


## 外部配置
Spring Boot 支持多种外部配置方式，这些方式优先级如下:
1. 命令行参数
2. 来自java:comp/env的JNDI属性
3. Java系统属性(System.getProperties())
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值
6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件
9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件
10. @Configuration注解类上的@PropertySource
11. 通过SpringApplication.setDefaultProperties指定的默认属性

## 参考
* https://www.cnblogs.com/lskreno/p/12222762.html
* https://juejin.cn/post/6990714491201650701
* https://segmentfault.com/a/1190000023471742