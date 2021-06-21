# 异常

## 异常体系
![Exception+20210621094516](https://raw.githubusercontent.com/loli0con/picgo/master/images/Exception%2B20210621094516.png%2B2021-06-21-09-45-18)

## 多异常捕获
一个catch块可以捕获多种类型的异常：
* 捕获多种类型的异常时，多种异常类型之间用竖线（|）隔开。
* 捕获多种类型的异常时，异常变量有隐式的final修饰，因此程序不能对异常变量重新赋值。

## 访问异常信息
所有的异常对象都包含了如下几个常用方法：
* getMessage()：返回该异常的详细描述字符串。
* printStackTrace()：将该异常的跟踪栈信息输出到标准错误输出。
* printStackTrace(PrintStream s)：将该异常的跟踪栈信息输出到指定输出流。
* getStackTrace()：返回该异常的跟踪栈信息。

## 自动关闭资源的try语句
在try关键字后紧跟一对圆括号，圆括号可以声明、初始化一个或多个资源，此处的资源指的是那些必须在程序结束时显式关闭的资源（比如数据库连接、网络连接等），try语句在该语句结束时自动关闭这些资源。

需要指出的是，为了保证try语句可以正常关闭资源，这些资源实现类必须实现AutoCloseable或Closeable接口，实现这两个接口就必须实现close()方法。

## checked异常
Java的异常被分为两大类：Checked异常和Runtime异常（运行时异常）。所有的RuntimeException类及其子类的实例被称为Runtime异常；不是RuntimeException类及其子类的异常实例则被称为Checked异常。
只有Java语言提供了Checked异常，其他语言都没有提供Checked异常。Java认为Checked异常都是可以被处理（修复）的异常，所以Java程序必须显式处理Checked异常。如果程序没有处理Checked异常，该程序在编译时就会发生错误，无法通过编译。

## throws声明
使用throws声明抛出异常的思路是，当前方法不知道如何处理这种类型的异常，该异常应该由上一级调用者处理；如果main方法也不知道如何处理这种类型的异常，也可以使用throws声明抛出异常，该异常将交给JVM处理。JVM对异常的处理方法是，打印异常的跟踪栈信息，并中止程序运行，这就是程序在遇到异常后自动结束的原因。
throws声明抛出只能在方法签名中使用，throws可以声明抛出多个异常类，多个异常类之间以逗号隔开。