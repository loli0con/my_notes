# 启动器
Spring Boot将所有的功能场景都抽取出来，做成一个个的starters(启动器)，只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。

## 项目结构
`starter --> autoconfigure --> springboot-starter`：
* starter没有任何代码，只是用来说明当前场景引入了哪些依赖，同时引入当前场景的真正自动配置包“autoconfigure”
* autoconfigure引入最底层的，每一个springboot模块都要使用的springboot-starter包。autoconfigure包中配置使用 META-INF/spring.factories 中 EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类。
  
![starter+20211013003738](https://raw.githubusercontent.com/loli0con/picgo/master/images/starter%2B20211013003738.png%2B2021-10-13-00-37-42)

## 制作步骤
1. 定义hello-spring-boot-starter启动器，引入hello-spring-boot-starter-autoconfigure自动配置包
2. 确定要复用的Service（提供服务/功能的组建），默认不要放在容器中（要使用自动配置类进行控制注入行为）
3. 定义属性类XxxProperties，使用@ConfigurationPropertie绑定配置文件中的配置项（提供给使用者来配置属性）
4. 定义自动配置类XxxAutoConfiguration，使用@Configuration注解表明该类是一个配置类，使用@ConditionalXxx条件注解来根据特点的条件控制配置行为是否生效（条件注解也可以放在配置类的方法上），使用@EnableConfigurationProperties注解将属性类注入到容器中
5. 在自动配置类中的方法上使用@Bean注解把要复用的Service注入到容器中
6. 在META-INF/spring.factories中org.springframework.boot.autoconfigure.EnableAutoConfiguration下写上自动配置类的全限定类名


## 参考
* https://www.bilibili.com/video/BV19K4y1L7MT?p=83
* https://www.yuque.com/atguigu/springboot/tmvr0e#efPjL