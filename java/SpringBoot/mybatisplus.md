# mybatis-plus
MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

[简介](https://baomidou.com/guide/#%E7%AE%80%E4%BB%8B)

## 快速开始 - demo
[快速开始](https://baomidou.com/guide/quick-start.html#%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B)

## 配置
[配置](https://baomidou.com/guide/config.html#%E9%85%8D%E7%BD%AE)

### datasource配置
```yml
# datasource
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mp # localhost:3306改成自己的数据库地址
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

### 实体类属性配置
```yml
mybatis-plus:
  global-config:
    db-config:
      # 表名前缀，设置后，如果数据库表名是tb_user, 那么实体类上可以不添加@TableName注解，不建议这里统一设置
      table-prefix: tb_
      # 全局默认主键类型，设置后，即可省略实体对象中的@TableId(type = IdType.AUTO)配置, 不建议这里统一设置
      id-type: auto 
  configuration:
    # 日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 打印sql
    # 设置sql返回字段驼峰转换
    map-underscore-to-camel-case: true

  # 使用原生mybatis以应对复杂的场景
  # 指定xml文件路径
  mapper-locations:
    - classpath:/mappers/*.xml
  # 返回值别名包路径
  type-aliases-package: cn.itcast.pojo
```


## 注解
[注解](https://baomidou.com/guide/annotation.html#%E6%B3%A8%E8%A7%A3)

### 自动映射
实体类的属性名和数据库的字段名自动映射：
1. 名称一样
2. 数据库字段使用_分割，实体类属性名使用驼峰名称

否则需要使用 @TableField(value="...") 或 @TableId(value="...") 指定映射关系

### 虚拟属性
在属性上配置@TableField(exist = false)，以此来忽略某个字段的查询和插入

## CRUD接口
[CRUD接口](https://baomidou.com/guide/crud-interface.html#crud-%E6%8E%A5%E5%8F%A3)

## 条件构造器
[条件构造器](https://baomidou.com/guide/wrapper.html#%E6%9D%A1%E4%BB%B6%E6%9E%84%E9%80%A0%E5%99%A8)

## 分页
[开启分页](https://baomidou.com/guide/page.html#%E5%88%86%E9%A1%B5%E6%8F%92%E4%BB%B6)

[分页查询](https://baomidou.com/guide/crud-interface.html#page)


### 自定义分页
使用原生的mybatis功能，加入Mybatis Plus的分页。

[自定义分页](https://baomidou.com/guide/page.html#xml-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%88%86%E9%A1%B5)

## 通用枚举
[通用枚举](https://baomidou.com/guide/enum.html#%E9%80%9A%E7%94%A8%E6%9E%9A%E4%B8%BE)

## service封装
Mybatis-Plus 为了开发更加快捷，对业务层也进行了封装，直接提供了相关的接口和实现类。我们在进行业务层开发时，可以继承它提供的接口和实现类，使得编码更加高效。

### 步骤
1. 定义接口继承IService
2. 定义实现类继承ServiceImpl\<Mapper，Entity>实现定义的接口

#### 定义Service接口
```java
public interface UserService extends IService<User> {}
```

#### 定义Service实现类
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    /**
     * test
     */
    public void test(){
        // this.baseMapper可以获取到userMapper对象，相当于已经注入进来了并且放到了this.baseMapper中;
        // 说明：Iservice中也包含了一些封装的CRUD方法
        UserMapper baseMapper = this.baseMapper;
    }
}
```

## 自动填充
我们在进行数据库表的更新/插入操作时，如果有创建时间、更新时间，每次插入都需要手动设置字段值，这就很麻烦。而Mybatis Plus提供了自动填充字段的设置。即给某些指定字段设置默认值。

[自动填充功能](https://baomidou.com/guide/auto-fill-metainfo.html)

### demo
#### 实体类
```java
@Data
public class User {
    ...
    // 创建时间，插入一条user数据时候，会自动设置创建时间到updated中
    @TableField(fill = FieldFill.INSERT)
    private Date created;
    
    // 修改时间，执行修改数据时候，会自动设置修改时间到updated中
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updated;
}
```

#### 编写handler
```java
package cn.itcast.handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.Date;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        // 参数1-字段名，参数2-默认值，参数3-固定写
        this.setFieldValByName("created", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        // 参数1-字段名，参数2-默认值，参数3-固定写
        this.setFieldValByName("updated", new Date(), metaObject);
    }
}
```

## 代码生成器
[代码生成器](https://baomidou.com/guide/generator.html#%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8)
### 依赖
```xml
<!--代码生成器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>

<!--模块引擎-->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.30</version>
</dependency>

<!--mybatis启动器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.0</version>
</dependency>
```

### 代码生成类
```java
package test;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.Scanner;

// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");  // 获取当前项目所在目录
        System.out.println(projectPath);
        gc.setOutputDir(projectPath + "/mybatisplus_code/src/main/java"); // 代码输出目录
        gc.setAuthor("itheima"); // 设置作者

        //设置包
        /* gc.setControllerName("Usercontroller");
        gc.setServiceName("UserService");
        gc.setEntityName("User");
        gc.setServiceImplName("UserServiceImpl");
        gc.setMapperName("UserDao"); */
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解

        // 设置全局配置
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/mp");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc); // 设置数据源

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("com.itheima");  // 包前缀
        pc.setModuleName(scanner("功能模块名")); // 设置功能模块名
        // 设置包配置
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        // strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
        // strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix("tb_"); // 设置表前缀
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());

        // 执行
        mpg.execute();

    }
}
```

### 项目结构
![mybatisplus+20210902154349](https://raw.githubusercontent.com/loli0con/picgo/master/images/mybatisplus%2B20210902154349.png%2B2021-09-02-15-43-50)

### 执行过程
![mybatisplus+20210902154507](https://raw.githubusercontent.com/loli0con/picgo/master/images/mybatisplus%2B20210902154507.png%2B2021-09-02-15-45-08)