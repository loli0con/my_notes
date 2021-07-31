# AOP
## 基本介绍
AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。

### 本质
AOP的本质是在一系列纵向的控制流程中，把那些相同的子流程提取成一个横向的面。将分散在主流程中相同的代码提取出来，然后在程序编译或运行时，将这些提取出来的切面代码应用到需要执行的地方。

### 织入的方式
在Java平台上，对于AOP的织入，有3种方式：
* 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入；
* 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”；
* 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入。

最简单的方式是第三种，Spring的AOP实现就是基于JVM的动态代理。由于JVM的动态代理要求必须实现接口，如果一个普通类没有业务接口，就需要通过CGLIB或者Javassist这些第三方库实现。

### 优点
利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

## AOP术语
* Pointcut(切入点，*在哪里注入*)：切入点指的是类或者方法名，满足某一规则的类或方法都是切入点，通过切入点表达式来制定规则。
* JoinPoint(连接点，*在切入点发生什么事情时候注入*)：在程序执行过程中的某个阶段点，连接点就是指主业务方法的调用，它是客观存在的。
* Advice(通知，*注入的时机*)：切入点处所要执行的程序代码，即切面类中要执行的公共方法。通知/拦截器的类型有：
  * 前置通知Before
  * 后置通知After
  * 异常通知AfterThrowing
  * 最终通知AfterReturning
  * 环绕通知Around
* Aspect(切面，*怎么被注入进去*)：切面指的是切入点(规则)和通知(织入方法)的类 = 切入点+通知
* Target(目标对象)：被代理/通知的对象，即真正执行业务的核心逻辑对象。比如动态代理案例中的明星，房东。
* Weaving(织入)：织入指的是把新增的功能用于目标对象，创建代理对象的过程，将切面整合到程序的执行流程的过程。
* Proxy(代理)：一个类被AOP织入增强后产生的结果类，即代理类，是客户端持有的增强后的对象引用。比如动态代理案例中的经纪人或中介。

其实，我们不用关心AOP创造的“术语”，只需要理解AOP本质上只是一种代理模式的实现方式，在Spring的容器中实现AOP特别方便。  

## xml配置
### 相关标签
![aop+20210731113325](https://i.loli.net/2021/07/31/K7ZuXa4xYqG2fkW.png)
* 配置AOP - <aop:config>
  * 配置切入点 - <aop:pointcut id="当前切入点的标识符" expression="切入点表达式">
  * (方式一)配置通知器 <aop:advisor advice-ref="引用通知类的id" pointcut-ref="引用切入点的id"/>
  * (方式二)配置切面 - <aop:aspect ref="引用切面类的id">
    * 配置通知 <aop:bofore method="切面类中的方法名" pointcut-ref="引用切入点的id">
    * 配置通知 <aop:after-returning method="切面类中的方法名" pointcut-ref="引用切入点的id">
    * 配置通知 <aop:after-throwing method="切面类中的方法名" pointcut-ref="引用切入点的id">
    * 配置通知 <aop:after method="切面类中的方法名" pointcut-ref="引用切入点的id">
    * 配置通知 <aop:around method="切面类中的方法名" pointcut-ref="引用切入点的id">

### Demo
```java
//方式一：通过 Spring API 实现AOP
/**
前置通知：org.springframework.aop.MethodBeforeAdvice
后置通知：
异常通知：org.springframework.aop.ThrowsAdvice
最终通知：org.springframework.aop.AfterReturningAdvice
环绕通知：org.aopalliance.intercept.MethodInterceptor
*/
class BeforeLog1 implements MethodBeforeAdvice {
   //method : 要执行的目标对象的方法
   //objects : 被调用的方法的参数
   //Object : 目标对象
   @Override
   public void before(Method method, Object[] objects, Object o) throws Throwable {
       System.out.println( o.getClass().getName() + "的" + method.getName() + "方法被执行了");
  }
}

//方式二：自定义类来实现AOP
class BeforeLog2 {
   public void printLog(){
       System.out.println("---------方法执行前---------");
  } 
}
```

```xml
<!--注册bean-->
<bean id="userService" class="com.demo.service.UserServiceImpl"/>
<bean id="beforeLog1" class="com.demo.log.BeforeLog1"/>
<bean id="beforeLog2" class="com.demo.log.BeforeLog2"/>

<!--aop的配置-->
<aop:config>
    <!--切入点 expression:表达式匹配要执行的方法-->
    <aop:pointcut id="pointcut" expression="execution(* com.demo.service.UserServiceImpl.*(..))"/>

    <!-- 方式一 -->
    <aop:advisor advice-ref="beforeLog1" pointcut-ref="pointcut"/>
    
    <!-- 方式二 -->
    <aop:aspect ref="beforeLog2">
        <aop:before method="printLog" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

### 切入点表达式
#### 切入点
|指示符|作用|
|---|---|
|execution|细粒度，精确到方法|
|within|粗粒度，精确到类|
|bean|粗粒度，精确到类(从容器中获取)|

```xml
<aop:pointcut id="切入点标识符" expression="bean(表达式) "/>
<aop:pointcut id="切入点标识符" expression="within(表达式) "/>
<aop:pointcut id="切入点标识符" expression="execution(表达式) "/>
```

#### 表达式expression
![aop+`访问修饰符? 返回类型 包名.类名.?方法名(参数类型) 抛出异常类型?`](https://i.loli.net/2021/07/31/PKowuZSsNymdQf3.png)
其中：
* ？表示出现0次或1次
* \* 表示通配符，可以匹配一个到多个参数
* .. 表示通配符，可以匹配零到多个参数
* 方法中参数个数
  * ()  没有参数
  * (*) 1个或多个参数
  * (..) 0个或多个参数
* 类全名的包（通配符）
  * .. 表示当前包和子包
* 逻辑运算符
  * &&(无意义，语法上存在，逻辑上不存在)
  *||
    * 示例：execution(* save(..))||execution(* update(..))
    * 解释：匹配方法名是save或update的方法
  * !
    * 示例：!execution(* save(..))
    * 解释：除了方法名是save的所有方法


### 通知advise
#### 通知类型
* 前置通知：在主业务方法前执行
* 后置通知：在主业务方法后执行
* 异常通知：在主业务方法抛出异常的时候执行
* 最终通知：无论主业务方法是否出现异常，都会执行。注：如果配置在后置通知的前面，会先执行最终通知
* 环绕通知：相当于上面所有通知的组合

```java
//伪代码
try{
  // 前置通知
  目标对象方法();
  //后置通知
}catch(Exception e){
  // 异常通知
}finally{
  // 最终通知
}
```

#### 环绕通知
##### 功能
* 可以环绕目标方法来执行
* 可以获取目标方法的各种信息
* 可以修改目标方法的参数和返回值
* 可以决定是否调用目标方法

##### ProceedingJoinPoint
Spring框架提供了ProceedingJoinPoint接口，作为环绕通知的参数。在环绕通知执行的时候，Spring框架会提供接口的对象，我们直接使用即可。

|ProceedingJoinPoint接口中方法|功能|
|---|---|
|Object[] getArgs()|获取目标方法的参数|
|proceed(Object[] args)|调用目标方法，如果没有执行这句话，目标方法不会执行|
|proceed()|调用目标方法，使用它原来的参数|
|getSignature()|获取目标方法其它的信息，如：类名，方法名等|

## 注解
AspectJ框架为AOP的实现提供了一套注解，用以取代applicationContext.xml文件中配置代码。

### 开启注解
在xml中添加如下标签以开启注解配置AOP的支持：
|\<aop:aspectj-autoproxy/>标签|作用|
|---|---|
|proxy-target-class属性|true：使用CGLIB进行代理。|
||false(默认)：使用JDK代理；如果目标类没有声明接口，则spring将自动使用CGLib动态代理。|

### 注解说明
|注解|说明|
|---|---|
|@Aspect|用在类上，表示这是一个切面类。<br />这个切面类要放到IoC容器中，所以类上还要加@Component注解(或者在xml中用\<bean>标签进行配置)|
|@Before|用在方法上，表示这是一个前置通知<br />value：指定切入点表达式|
|@AfterReturning|用在方法上，表示这是一个最终通知<br />value：指定切入点表达式|
|@AfterThrowing|用在方法上，表示这是一个异常通知<br />value：指定切入点表达式|
|@After|用在方法上，表示这是一个后置通知。<br />value：指定切入点表达式|
|@Around|用在方法上，表示这是一个环绕通知<br />value：指定切入点表达式|
|@Pointcut|用在方法上，用来定义切入点表达式<br />方法名：随意起<br />返回值：void<br />方法体：为空|

### Demo
```java
@Component
@Aspect  //这是一个切面类
public class LogAspect {
    //定义切入点函数
    @Pointcut("execution(* com.demo.service..*(..))")
    public void pt() {

    }

    @Before("pt()")  //写方法的名字
    public void before() {
        System.out.println("---前置通知---");
    }

    @After("execution(* com.demo.service.UserServiceImpl.*(..))") //写表达式
    public void after() {
        System.out.println("---最终通知---");
    }
}
```
