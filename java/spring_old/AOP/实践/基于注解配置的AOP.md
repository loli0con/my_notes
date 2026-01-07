# 基于注解配置的AOP
Spring的AOP也提供了注解方式配置，使用相应的注解替代之前的xml配置。

## 步骤
主要配置三部分:
1. 目标类被Spring容器管理
2. 通知类被Spring容器管理
3. 配置aop：通知与切点的织入(切面)

```Java
@Component("target") // 目标类被Spring容器管理
public class TargetImpl implements Target{
    public void show() {
    System.out.println("show Target running...");
    }
}

@Aspect // 配置aop第一步
@Component // 通知类被Spring容器管理
public class AnnoAdvice {
    @Around("execution(* com.itheima.aop.*.*(..))") //配置aop第二步
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前通知...");
        joinPoint.proceed();
        System.out.println("环绕后通知...");
    }
}
```

## 启用注解扫描
在配置中启用对@Aspect、@Around等aspectj包下注解的扫描。

#### 方式1 - 针对XML配置文件
```XML
<aop:aspectj-autoproxy/>
```

#### 方式2 - 针对配置类 
```Java
@Configuration
@ComponentScan("com.itheima.aop")
@EnableAspectJAutoProxy //注解支持
public class ApplicationContextConfig { }
```


## 原理

### 过程
从aspectj-autoproxy标签的解析器入手：
```Java
// 注册aspectj-autoproxy标签的解析器
this.registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
```

AspectJAutoProxyBeanDefinitionParser代码内部，最终也是执行了和xml方式AOP一样的代码：
```Java
registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source)
```

如果使用的是核心配置类的话，查看@EnableAspectJAutoProxy源码，使用的也是@Import导入相关解析类：
```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({AspectJAutoProxyRegistrar.class})
public @interface EnableAspectJAutoProxy {
    boolean proxyTargetClass() default false;
    boolean exposeProxy() default false;
}
```

使用@Import导入的AspectJAutoProxyRegistrar源码，一路追踪下去，最终还是注册了 AnnotationAwareAspectJAutoProxyCreator这个类。

![基于注解配置的AOP+20231116224604](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8E%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116224604.png%2B2023-11-16-22-46-05)