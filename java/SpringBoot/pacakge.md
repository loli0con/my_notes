# 打包

## 配置
### pom
```xml
<!-- 配置打包方式(默认为jar) -->
<!-- <packaging>jar</packaging> -->
<packaging>war</packaging>


<!-- 指定scope为provided: 代表打包时，不需要它的依赖jar包 -->
<!-- 这样在打war包时，可以排除内嵌的tomcat（因为我们有自己的tomcat，有自己的运行平台/环境） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

<build>

    <!-- 指定最终打成war的项目名称 -->
    <finalName>ROOT</finalName>

    <plugins>
        <!-- 配置spring-boot的maven插件
                1. 用它可以运行spring-boot项目
                2. 需要用它构建打jar、war资料
             -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 应用入口类
注意：该类是在打war包时配置的。
```java
/** web应用入口，继承SpringBootServletInitializer 即 相当于web.xml */
public class WebServletInitializer extends SpringBootServletInitializer {
    
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        // 设置启动类
        builder.sources(MyApplication.class);
        // 返回spring应用构建对象
        return builder;
    }
}
```

## 命令
```sh
# 清理、打包
mvn clean package
# 清理、打包 跳过测试
mvn clean package -Dmaven.test.skip=true

java -jar xxx.jar
```

## 部署

## 运行

## 测试