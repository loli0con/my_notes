# Servle开发示例

## 创建Maven项目
![20260104171310](https://raw.githubusercontent.com/loli0con/picgo/master/20260104171310.png)

![20260104171519](https://raw.githubusercontent.com/loli0con/picgo/master/20260104171519.png)

配置pom.xml如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>my-web-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!-- 打包类型为war，全称Java Web Application Archive，JavaWeb应用归档 -->
    <packaging>war</packaging>

    <dependencies>
        <!--引入Servlet API-->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>5.0.0</version>
            <!-- provided表示编译时使用（用于代码提示补全检查等），但不会打包到war中，Tomcat程序中已经自带了该依赖 -->
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <properties>
        <!--Maven自动配置编译器版本为Java17-->
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <build>
        <plugins>
            <!--使用新版本的打包插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.5.1</version>
            </plugin>
        </plugins>
    </build>

</project>
```

## 编写Servlet
![20260104172744](https://raw.githubusercontent.com/loli0con/picgo/master/20260104172744.png)

## 运行调试
![20260104172931](https://raw.githubusercontent.com/loli0con/picgo/master/20260104172931.png)

![20260104173316](https://raw.githubusercontent.com/loli0con/picgo/master/20260104173316.png)

![20260104173451](https://raw.githubusercontent.com/loli0con/picgo/master/20260104173451.png)

![20260104174559](https://raw.githubusercontent.com/loli0con/picgo/master/20260104174559.png)

![20260104180707](https://raw.githubusercontent.com/loli0con/picgo/master/20260104180707.png)

## WEB-INF
![20260104180433](https://raw.githubusercontent.com/loli0con/picgo/master/20260104180433.png)

## 使用xml配置Servlet
![20260104182230](https://raw.githubusercontent.com/loli0con/picgo/master/20260104182230.png)

## 静态资源
![20260104220917](https://raw.githubusercontent.com/loli0con/picgo/master/20260104220917.png)