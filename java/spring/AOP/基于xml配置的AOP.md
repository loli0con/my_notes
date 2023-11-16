# 基于xml配置的AOP

## 步骤
xml方式配置AOP的步骤:
1. 导入AOP相关坐标
2. 准备目标类、准备增强类，并配置给Spring管理
3. 配置切点表达式(哪些方法被增强)
4. 配置织入(切点被哪些通知方法增强，是前置增强还是后置增强)

### 导入AOP相关坐标
```XML
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>
```
Spring-context坐标下已经包含spring-aop的包了，所以就不用额外导入了。

### 准备目标类、准备增强类，并配置给Spring管理
```Java
// 目标接口
public interface UserService {
    void show1();
    void show2();
}

// 目标类
public class UserServiceImpl implements UserService {
    public void show1() {
        System.out.println("show1...");
    }
    
    public void show2() {
        System.out.println("show2...");
    }
}

// 增强类
public class MyAdvice {
    public void beforeAdvice(){
        System.out.println("beforeAdvice");
    }

    public void afterAdvice(){
        System.out.println("afterAdvice");
    }
}
```

```XML
<!-- 配置目标类,内部的方法是连接点 -->
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl" />

<!-- 配置通知类,内部的方法是增强方法 -->
<bean id="myAdvice" class="com.itheima.advice.MyAdvice" />
```

### 配置切点表达式 & 配置织入
```XML
<aop:config>
    <!--配置切点表达式,对哪些方法进行增强-->
    <aop:pointcut id="myPointcut" expression="execution(void com.itheima.service.impl.UserServiceImpl.show1())"/>
    
    <!--切面=切点+通知-->
    <aop:aspect ref="myAdvice">
        <!--指定前置通知方法是beforeAdvice-->
        <aop:before method="beforeAdvice" pointcut-ref="myPointcut"/>
        <!--指定后置通知方法是afterAdvice-->
        <aop:after-returning method="afterAdvice" pointcut-ref="myPointcut"/>
    </aop:aspect>
</aop:config>
```

## 通知类型
AspectJ的通知有五种类型：
|通知名称|配置方式|执行时机|
|---|---|---|
|前置通知|`<aop:before>`|目标方法执行之前执行|
|后置通知|`<aop:after-returning>`|目标方法执行之后执行，目标方法异常时，不执行|
|环绕通知|`<aop:around>`|目标方法执行前后执行，目标方法异常时，环绕后方法不执行|
|异常通知|`<aop:after-throwing>`|目标方法抛出异常时执行|
|最终通知|`<aop:after>`|不管目标方法是否有异常，最终都会执行|

### Demo
```Java
// 环绕通知
public void around(ProceedingJoinPoint joinPoint) throws Throwable {
    //环绕前
    System.out.println("环绕前通知");
    //目标方法
    joinPoint.proceed();
    //环绕后
    System.out.println("环绕后通知");
}

// 异常通知
public void afterThrowing(){
    System.out.println("目标方法抛出异常了，后置通知和环绕后通知不在执行");
}

// 最终通知
public void after(){
    System.out.println("不管目标方法有无异常，我都会执行");
}
```

```XML
<!-- 环绕通知 -->
<aop:around method="around" pointcut-ref="myPointcut"/>

<!-- 异常通知 -->
<aop:after-throwing method="afterThrowing" pointcut-ref="myPointcut"/>

<!-- 最终通知 -->
<aop:after method="after" pointcut-ref="myPointcut"/>
```

## advisor
AOP的另一种配置方式，该方式需要通知类实现Advice的子功能接口：
```Java
public interface Advice { }
```

```Java
public class MyAdvices implements MethodBeforeAdvice, AfterReturningAdvice, MethodInterceptor {
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("This is before Advice ...");
    }

    @Override
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("This is afterReturn Advice ...");
    }
    
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {  
        System.out.println("前置逻辑功能...");
        //执行目标方法
        Object invoke = methodInvocation.getMethod().invoke(
            methodInvocation.getThis(),
            methodInvocation.getArguments()
        );
        System.out.println("后置逻辑功能...");
        return invoke;
    }
}
```

```XML
<aop:config>
    <!-- 将通知和切点进行结合 -->
    <aop:advisor advice-ref="myAdvices" pointcut="execution(void com.itheima.aop.TargetImpl.show())"/>
</aop:config>
```



## 原理

### 过程剖析
通过xml方式配置AOP时，引入了AOP的命名空间。在spring-aop包下的META-INF去找相应的spring.handlers文件。

![基于xml配置的AOP+20231116163408](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116163408.png%2B2023-11-16-16-34-10)

![基于xml配置的AOP+20231116163636](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116163636.png%2B2023-11-16-16-36-39)

最终加载的是AopNamespaceHandler，该Handler的init方法中注册了config标签对应的解析器：
```Java
this.registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
```

以ConfigBeanDefinitionParser作为入口进行源码剖析，最终会注册一个AspectJAwareAdvisorAutoProxyCreator进入到Spring容器中。

![基于xml配置的AOP+20231116164655](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116164655.png%2B2023-11-16-16-46-58)

AspectJAwareAdvisorAutoProxyCreator的上上级父类AbstractAutoProxyCreator中的
postProcessAfterInitialization方法：
```Java
//Spring容器中，所有的Bean都会经历该生命周期。其中方法形参bean为目标对象，即所有的Bean
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            //如果需要被增强，则wrapIfNecessary方法最终返回的就是一个Proxy对象
            return this.wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

对wrapIfNecessary进行剖析：
![基于xml配置的AOP+20231116165505](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116165505.png%2B2023-11-16-16-55-08)

对createProxy进行剖析：
![基于xml配置的AOP+20231116165726](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116165726.png%2B2023-11-16-16-57-29)

对getProxy进行剖析：
![基于xml配置的AOP+20231116170023](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116170023.png%2B2023-11-16-17-00-26)

最底层的getProxy有两个实现：
![基于xml配置的AOP+20231116170205](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116170205.png%2B2023-11-16-17-02-07)

最终生成的Proxy对象：
![基于xml配置的AOP+20231116170653](https://raw.githubusercontent.com/loli0con/picgo/master/images/%E5%9F%BA%E4%BA%8Exml%E9%85%8D%E7%BD%AE%E7%9A%84AOP%2B20231116170653.png%2B2023-11-16-17-06-57)

### 过程总结
1. 首先对`xmlns:aop`进行自定义命名空间解析，为`<aop:config>`标签注册解析器
2. 注册解析器时，向Spring容器中注入`AspectJAwareAdvisorAutoProxyCreator`
3. `AspectJAwareAdvisorAutoProxyCreator`的本质是一个`BeanPostProcessor`
4. 在该`BeanPostProcessor`的`after`方法中生成bean代理对象
5. 最终把这个生成的bean代理对象注入到Spring容器中


### 动态代理的实现
AopProxy接口有两个实现类，这两种都是动态生成代理对象的方式，一种就是基于JDK的，一种是基于Cglib的：
|代理技术|使用条件|配置方式|
|---|---|---|
|JDK 动态代理技术|目标类有接口，是基于接口动态生成实现类的代理对象|目标类有接口的情况下，默认方式|
|Cglib 动态代理技术|目标类无接口且不能使用final修饰，是基于被代理对象动态生成子对象为代理对象|目标类无接口时，默认使用该方式;目标类有接口时，手动配置`<aop:config proxy-target-class=“true”>`强制使用Cglib方式|


#### Demo - Cglib
```Java
Target target = new Target();//目标对象
Advices advices = new Advices();//通知对象
Enhancer enhancer = new Enhancer();//增强器对象
enhancer.setSuperclass(Target.class);//增强器设置父类
//增强器设置回调
enhancer.setCallback((MethodInterceptor )(o, method, objects, methodProxy) -> {
    advices.before();
    Object result = method.invoke(target, objects); advices.afterReturning();
    return result;
});
//创建代理对象
Target targetProxy = (Target) enhancer.create();
//测试
String result = targetProxy.show("xxx");
```
