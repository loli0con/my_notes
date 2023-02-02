# 动态代理

## 代理模式
代理模式是一种结构型设计模式：给某一个对象提供一个代理，并由代理对象来控制对真实对象的访问。

代理模式角色分为3种：
1. Subject（抽象主题角色）：为了让客户端能够一致性地对待真实对象和代理对象而在引入的抽象层，定义代理类和真实主题的公共对外方法，也是代理类代理真实主题的方法
2. RealSubject（真实主题角色）：真正实现业务逻辑的类
3. Proxy（代理主题角色）：用来代理和封装真实主题

![DynamicProxy+20230202113731](https://raw.githubusercontent.com/loli0con/picgo/master/images/DynamicProxy%2B20230202113731.png%2B2023-02-02-11-37-31)

根据字节码的创建时机来分类，可以分为静态代理和动态代理：
1. 静态代理就是在程序运行前就已经存在代理类的字节码文件，代理类和真实主题角色的关系在运行前就确定了
2. 动态代理的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以在运行前并不存在代理类的字节码文件

## JDK动态代理
在Java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口，通过使用Proxy类和InvocationHandler接口可以生态JDK动态代理类或动态代理对象。

![reflection+20210606174524](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606174524.png%2B2021-06-06-17-45-25)

动态代理实际上是JVM在运行期动态创建class字节码并加载的过程。

在运行期动态创建一个interface实例的方法如下：
1. 定义一个InvocationHandler实例，它负责实现接口的方法调用；
2. 通过Proxy.newProxyInstance()创建interface实例，它需要3个参数：
   1. 使用的ClassLoader，通常就是接口类的ClassLoader；
   2. 需要实现的接口数组，至少需要传入一个接口进去；
   3. 用来处理接口方法调用的InvocationHandler实例。
3. 将返回的Object强制转型为接口。

一个最简单的动态代理实现如下：
```Java
public class Main {
    public static void main(String[] args) {
        // InvocationHandler：调用处理器。该接口只有一个invoke方法。
        InvocationHandler handler = new InvocationHandler() {
            // 通过实现invoke方法，可以控制“代理对象proxy 传入 args参数列表 调用 method方法”的过程
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method);
                if (method.getName().equals("morning")) {
                    System.out.println("Good morning, " + args[0]);
                }
                return null;
            }
        };
        // 创建一个代理对象
        Hello hello = (Hello) Proxy.newProxyInstance(
            Hello.class.getClassLoader(), // 传入ClassLoader
            new Class[] { Hello.class }, // 传入要实现的接口
            handler); // 传入处理调用方法的InvocationHandler

        // 代理对象hello 传入 参数列表["Bob"] 调用 morning方法
        hello.morning("Bob");
    }
}

interface Hello {
    void morning(String name);
}
```

## 推荐阅读
[Java动态代理详解](https://juejin.cn/post/6844903744954433544)

## 
