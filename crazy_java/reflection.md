# 反射
反射是指程序在运行期可以拿到一个对象/类的所有信息。

![reflection+Class](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2BClass.png%2B2021-06-06-18-54-57)

> 🌝拿到所有信息以后，基本就可以为所欲为了。

## 获取Class对象
每个类被加载之后，系统就会为该类生成一个对应的Class对象，通过该Class对象就可以访问到JVM中的这个类。有三种方式获取Class对象：
1. 使用Class类的forName(String clazzName)静态方法，该方法需要传入字符串参数，该字符串参数的数值是某个类的全限定类名
2. 调用某个类的class属性来获取该类对应的Class对象
3. 调用某个对象的getClass()方法

一旦获取了某个类对应的Class对象之后，程序就可以调用Class对象的方法来获得该对象和该类的真实信息了。

## 从Class中获取信息
Class类提供了大量的实例方法来获取该Class对象所对应类的详细信息。

### 构造器
![reflection+20210606151659](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606151659.png%2B2021-06-06-15-16-59)

### 方法
![reflection+20210606151730](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606151730.png%2B2021-06-06-15-17-31)

### 成员变量
![reflection+20210606151809](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606151809.png%2B2021-06-06-15-18-10)

### 注解
![reflection+20210606151903](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606151903.png%2B2021-06-06-15-19-04)

### 内部类
![reflection+20210606151925](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606151925.png%2B2021-06-06-15-19-26)

### 修饰符 所在包 类名
![reflection+20210606152110](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606152110.png%2B2021-06-06-15-21-11)

### 类信息
![reflection+20210606152151](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606152151.png%2B2021-06-06-15-21-52)

### Executable剖析
![reflection+20210606162728](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606162728.png%2B2021-06-06-16-27-30)

#### 注意
当isNamePresent返回为ture时，getName、getParameterizedType、getType才能得到相应信息。

## 生成并操作对象
### 访问权限检查
在访问类成员时，需要具备权限，通常无法访问被private修饰的成员。

类成员（Method、Constructor、Field）的父类AccessibleObject有一个setAccessible(boolean flag)方法，类成员可以调用该方法来取消访问权限检查，通过反射来访问private成员。

### 创建对象
使用Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建Class对象对应类的实例。

### 调用方法
通过Class对象获取指定的Methond对象，通过该Method来调用它对应的方法。在Method里包含一个invoke方法，该方法的签名如下：  
Object invoke(Object obj, Object... args)，该方法中的obj是执行该方法的主调，后面的args是执行该方法时传入该方法的实参。

### 访问成员变量值
通过Class对象获取指定的Filed对象，再调用Field对象的方法来读取和设置成员变量值。


### 操作数组
在java.lang.reflect包下还提供了一个Array类，Array对象可以代表所有的数组。程序可以通过使用Array来动态地创建数组，操作数组元素等。

可以创建多维数组：
`static Object newInstance(Class<?> componentType, int... length)`：创建一个具有指定的元素类型、指定维度的新数组。


## JDK动态代理
在Java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口，通过使用这个类和接口可以生态JDK动态代理类或动态代理对象。

Proxy提供了用于创建动态代理类和代理对象的静态方法，它也是所有动态代理类的父类。

![reflection+20210606174524](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2B20210606174524.png%2B2021-06-06-17-45-25)

**请阅读[Java动态代理详解](https://juejin.cn/post/6844903744954433544)。**

## 获取泛型信息
[ParameterizedType详解](https://blog.csdn.net/LVXIANGAN/article/details/94836504)

## 后记
[Type详解](https://blog.csdn.net/u010884123/article/details/78189395)