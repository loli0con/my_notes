# 类的加载
系统可能在第一次使用某个类时加载该类，也可能采用预加载机制来加载某个类。

## JVM进程
当调用java命令运行某个Java程序时，该命令将会启动一个Java虚拟机进程，即JVM进程。  
同一个JVM的所有线程、所有变量都在都处于同一个进程里，它们都是用该JVM进行的内存区。

当出现以下几种情况时，JVM进程将被终止：
* 程序运行到最后正常结束
* 当程序运行到使用`System.exit()`或`Runtime.getRuntine().exit()`代码处结束程序
* 程序执行过程中遇到未捕获的异常或者错误而结束
* 程序所在平台强制结束了JVM进程

## 类加载/类初始化
类加载（或者叫类初始化）有三个步骤：
1. 加载
2. 连接
3. 初始化

在对当前类执行上述三个步骤之前，系统会先对当前类的直接父类执行上述三个步骤，以此类推。

### 加载
类的加载是指将类的class文件读入内存，并为之创建一个java.lang.Class对象。

### 连接
连接阶段负责把类的二进制数据合并到JRE中，又可以分为如下三个阶段：
1. 验证：校验被加载的类是否有正确的内部结构，并和其他类协调一致
2. 准备：为类变量分配内存，并设置默认初始化值
3. 解析：将类的二进制数据中的符号引用替换成直接引用

### 初始化
类的初始化主要是对类变量进行初始化，有两种方式：
1. 声明类变量时指定初始值
2. 静态初始化块里为类变量指定初始化

JVM会按照这些语句在程序中的排列顺依次执行它们。

### 加载时机
![class_load+20210606102542](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606102542.png%2B2021-06-06-10-25-43)

当某个类变量使用了final修饰，而且它的值可以在编译时就确定下来，那么这个类变量相当于“宏变量”，因此即使程序使用该类变量，也不会导致该类的初始化。

## 加载器
类的加载由类加载器完成，类加载器通常由JVM提供（称为“系统类加载器”），开发者也可以创建自己的类加载器（称为“用户类加载器”）。

![class_load+20210606120259](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606120259.png%2B2021-06-06-12-03-02)

### 唯一标识
在JVM中，一个类用其全限定类名和其类加载器作为唯一标识。

### 来源
使用不同的类加载器，可以从不同来源加载类的二进制数据，通常由如下几种来源：
* 本地的class文件
* JAR包中的class文件
* 网络里的class文件
* Java源文件

### 初始类加载器
![class_load+20210606110216](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606110216.png%2B2021-06-06-11-02-16)

Bootstrap ClassLoader 被称为引导（也称为原始或根）类加载器，它负责加载Java核心类。

![class_load+20210606120826](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606120826.png%2B2021-06-06-12-08-27)

JVM的根类加载器并不是Java实现的，而且由于程序通常无须访问根类加载器，因此访问拓展类加载器的父类加载器时返回null。

### 加载机制
![class_load+20210606110443](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606110443.png%2B2021-06-06-11-04-44)

类加载器之间的父子关系并不是类继承上的父子关系，这里的父子关系是类加载器实例之间的关系。

### 加载步骤
![class_load+20210606115117](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606115117.png%2B2021-06-06-11-51-18)

### 用户类加载器
![class_load+20210606120530](https://raw.githubusercontent.com/loli0con/picgo/master/images/class_load%2B20210606120530.png%2B2021-06-06-12-05-31)