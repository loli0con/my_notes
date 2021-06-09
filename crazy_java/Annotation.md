# 注解
Annotation
![Annotation+20210609103638](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609103638.png%2B2021-06-09-10-36-39)
## 基本概念

### 定义
注解是代码里的**特殊标记**，这些标记可以在*编译、类加载、运行时*被读取，并执行**相应操作**。

注解也是一个**接口**，可以通过反射来获取它。

### 特殊标记
通过使用注解，可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充信息。从某些方面来看，注解就像修饰符一样，可用于修饰包、类、构造器、方法、成员变量、参数、局部变量的声明。

注解提供了一种为程序元素设置[元数据](http://www.ruanyifeng.com/blog/2007/03/metadata.html)的方法。

注解也不会影响代码的运行。

### 相应操作
代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证或进行部署。

如果希望让程序中的注解在运行时发挥一定的作用，只有通过某种配套工具对注解中的信息进行访问和处理，这种工具被统称为APT（Annotation Processing Tool）。

## 基本Annotation
在java.lang下，...
### @Override
限定重写父类方法：它用于强制指定某个子类的方法必须覆盖父类的方法，否则会编译出错。

### @Deprecated
标记已过时：它用于表示某个程序元素（类、方法等）已过时，当其他程序使用已过时的类、方法时，编译器会给出警告。

### @SuppressWarnings
抑制编译器警告：它会让被修饰的程序元素（以及该程序元素中的所有子元素）取消现实指定的编译器警告。

### @SafeVarargs
抑制堆污染警告：它会让被修饰的程序元素（方法或构造器）取消堆污染警告。

#### 堆污染
当把一个不带泛型的对象赋给一个带泛型的变量时，往往会发生堆污染。

> TODO

### @FunctionalInterface
函数式接口：它指定的某个接口必须是函数式接口。

## 元Annotation
在java.lang.annotation下，它们用于修饰其他Annotation...

### @Retention
@Retention只能用于修饰Annotion定义，用于指定被修饰的Annotation可以保留多长时间。它包含一个RetentionPolicy类型的value成员变量，所以使用它时必须为该value成员变量指定值。  
value成员变量的值只能是如下三个：
* RetentionPolicy.CLASS：编译器将把Annotation记录在class文件中。当运行Java程序时，JVM不可获取Annotation信息。这是默认值。
* RetentionPolicy.RUNTIME：编译器将把Annotation记录在class文件中。当运行Java程序时，JVM也可获取Annotation信息，程序可通过反射获得该Annotation信息。
* RetentionPolicy.SOURCE：Annotation只保留在源代码中，编译器直接丢弃这种Annotation。

### @Target
@Target只能用于修饰Annotion定义，用于指定被修饰的Annotation能用于修饰哪些程序单元。它包含一个名为value的成员变量，该成员变量的值只能是如下几个：
![Annotation+20210609092220](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609092220.png%2B2021-06-09-09-22-21)

#### 增强
修饰任何用到类型的地方！

### @Documented
@Documented用于指定被该元Annotation修饰的Annotation类将被javadoc工具提取成文档，如果定义Annotation类时使用了@Documented修饰，则所有使用该Annotation修饰的程序元素的API文档中将会包含该Annotation说明。
![Annotation+20210609093357](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609093357.png%2B2021-06-09-09-33-58)

### @Inherited
@Inherited元Annotation指定被它修饰的Annotation将具有继承性——如果某个类使用了@Xxx注解（定义该Annotation时使用了@Inherited修饰）修饰，则其子类将自动被@Xxx修饰。

### @Repeatable
```Java
// 定义一个普通的注解
@Retention(RetentionPolicy.RUNTIME)
public @interface FkTag{
    String name() defalut "啦啦啦";
    int age();
}
```
普通注解（FkTag）默认不能作为重复注解使用，如果使用两个以上的普通注解修饰同一个程序元素，编译器会报错。为了把普通注解改造成重复注解，需要使用@Repeatable修饰该注解，使用@Repeated时必须为value成员变量指定值，该成员变量的值是一个“容器”注解——该“容器注解”可包含多个普通注解，因此需要定义如下的“容器”注解。
```Java
// 定义一个"容器"的注解
@Retention(RetentionPolicy.RUNTIME)
public @interface FkTags{
    // 定义value成员变量，该成员变量可接受多个@FkTag注解
    FkTag[] value();
}
```
接下来程序可在定义@FkTag注解时添加如下修饰代码：
```Java
@Repeatable(FkTags.class)  // 重点是这行！！！
/**
  定义FkTag注解的代码
 */
```

#### 注解的保存期
![Annotation+20210609103122](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609103122.png%2B2021-06-09-10-31-23)


## 自定义Annotation
### 语法
定义新的Annotation类型使用@interface关键字，它继承了java.lang.Annotation接口。Annotation的成员变量在Annotation定义中以无形参的方法形式来声明，其方法名和返回值定义了该成员变量的名字和类型。
```Java
public @interface MyTag{
    String auther() default "loli0con";
    int version();
}
```
一旦在Annotation里定义了成员变量之后，使用该Annotation时就应该为该Annotation的成员变量指定值。

也可以在定义Annotation成员变量时为其指定初始值（默认值），指定成员变量的初始值可使用default关键字。  
如果为Annotation的成员变量指定了默认值，使用该Annotation时则可以不为这些成员变量指定值，而是直接使用默认值。当然也可以在使用时为成员变量指定值，则默认值不会起作用。

### 分类
根据Annotation是否可以包含成员变量，可以把Annotation分为如下两类：
* 标记Annotation：没有定义成员变量的Annotation类型被称为标记。这种Annotation利用自身的存在与否来提供信息。
* 元数据Annotation：包含成员变量的Annotation，因为它们可以接受更多的元数据，所以也被称为元数据Annotation。

### 提取
使用Annotation修饰了类、方法、成员变量等成员之后，这些Annotation不会自己生效，必须由开发者提供相应的工具来提取并处理Annotation信息。

![Annotation+20210609100312](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609100312.png%2B2021-06-09-10-03-12)

java.lang.reflect包主要包含一些实现反射功能的工具类，这些反射API具备读取运行时Annotation的能力。只有当定义Annotation时使用了@Retention（RetentionPolicy.RUNTIME）修饰，该Annotation才会在运行时可见，JVM才会在装在*.class文件时读取保存在class的Annotation。

![Annotation+20210609100627](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609100627.png%2B2021-06-09-10-06-28)

### 使用
> TODO

## Type Annotation
### 定义
![Annotation+20210609104612](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609104612.png%2B2021-06-09-10-46-13)

### 示例
![Annotation+20210609104851](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609104851.png%2B2021-06-09-10-48-54)

## APT
![Annotation+20210609105124](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609105124.png%2B2021-06-09-10-51-25)

![Annotation+20210609105213](https://raw.githubusercontent.com/loli0con/picgo/master/images/Annotation%2B20210609105213.png%2B2021-06-09-10-52-14)

## 后记
[Element详解](https://www.jianshu.com/p/899063e8452e)