# 反射
反射是指程序在运行期可以拿到一个对象/类的所有信息。

![reflection+Class](https://raw.githubusercontent.com/loli0con/picgo/master/images/reflection%2BClass.png%2B2021-06-06-18-54-57)

> 🌝拿到所有信息以后，基本就可以为所欲为了。

## Class类
除了int等基本类型外，Java的其他类型全部都是**class**（包括interface），**class**（包括interface）的本质是**数据类型（Type）**。
* `String` -> **String.class** -> 是一种数据类型
* `Object` -> **Object.class** -> 是一种数据类型
* `Runnable` -> **Runnable.class** -> 是一种数据类型
* `Exception` -> **Exception.class** -> 是一种数据类型
* `Class` -> **Class.class** -> 是一种数据类型

**class**是由JVM在执行过程中动态加载的，JVM在第一次读取到一种**class**类型时，将其加载进内存。每加载一种**class**，JVM就为其创建一个`Class`类型的实例，并关联起来。注意：这里的`Class`类型是一个名叫`Class`的**class**。所以，JVM持有的每个`Class`实例都指向一个数据类型（class或interface）。

由于JVM为每个加载的**class**创建了对应的`Class`实例，并在实例中保存了该**class**的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的**class**的所有信息。

这种通过`Class`实例获取**class**信息的方法称为反射（Reflection）。

### 数组
数组（例如String[]）也是一种类，而且不同于String.class，它的类名是[Ljava.lang.String。

### 基本类型
JVM为每一种基本类型如int也创建了Class实例，通过int.class访问。

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
当isNamePresent返回为true时，getName、getParameterizedType、getType才能得到相应信息。

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



## 获取泛型信息
[ParameterizedType详解](https://blog.csdn.net/LVXIANGAN/article/details/94836504)

## 后记
[Type详解](https://blog.csdn.net/u010884123/article/details/78189395)