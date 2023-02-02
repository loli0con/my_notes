# 异常

## 异常处理机制
Java的异常处理机制可以让程序具有极好的**容错**性，让程序更加**健壮**。当程序运行出现意外情形时，系统会自动生成一个Exception对象来通知程序（该异常对象被提交给Java运行时环境，这个过程被称为*抛出(throw)异常*），从而实现将“业务功能实现代码”和“错误处理代码”**分离**，提供更好的可读性。

当Java运行时环境收到异常对象时，会寻找能处理该异常对象的catch块，如果找到合适的catch块，则把该异常对象交给该catch块处理，这个过程被称为*捕获(catch)异常*；如果Java运行时环境找不到捕获异常的catch块，则运行时环境终止，Java程序也将退出。

## 异常体系
![Exception+20210621094516](https://raw.githubusercontent.com/loli0con/picgo/master/images/Exception%2B20210621094516.png%2B2021-06-21-09-45-18)

Java把所有的非正常情况分成两种：异常（Exception）和错误（Error），它们都继承Throwable父类。

### 错误
Error错误，一般是指与虚拟机相关的问题，如系统崩溃、虚拟机错误、动态链接失败等，这种错误无法恢复或不可能捕获，将导致应用程序中断。

通常应用程序无法处理这些错误，因此应用程序不应该试图使用catch块来捕获Error对象。

### 异常
Java将异常分为两种，**Checked异常**和**Runtime异常**。  
Java认为Checked异常都是可以在编译阶段被处理的异常，所以它强制程序处理所有的Checked异常；而Runtime异常则无需处理。

所有的RuntimeException类及其子类的实例被称为Runtime异常；不是RuntimeException类及其子类的异常实例则被称为Checked异常。

## 抛出异常
抛出异常分两步：
1. 创建某个Exception的实例；
2. 用throw语句抛出。

实际上，绝大部分抛出异常的代码都会合并写成一行：
```Java
throw new RuntimeException();
```

如果一个方法捕获了某个异常后，又在catch子句中抛出新的异常，就相当于把抛出的异常类型“转换”了。为了能追踪到完整的异常栈，在构造新的异常的时候，把原始的Exception实例传进去，新的Exception就可以持有原始Exception信息。

在代码中获取原始异常可以使用Throwable.getCause()方法。如果返回null，说明已经是“根异常”了。

## 捕获异常
捕获异常使用`try...catch`语句，把可能发生异常的代码放到try {...}中，然后使用catch捕获对应的Exception及其子类。

RuntimeException无需强制捕获，非RuntimeException（Checked Exception）需强制捕获，或者用throws声明；只要方法声明了Checked Exception，不在调用层捕获，也必须在更高的调用层捕获。

### throws声明
在方法定义的时候，使用`throws Xxx`声明该方法可能抛出的异常类型。使用throws声明抛出异常的思路是，当前方法不知道如何处理这种类型的异常，该异常应该由上一级调用者处理；如果main方法也不知道如何处理这种类型的异常，也可以使用throws声明抛出异常，该异常将交给JVM处理。JVM对异常的处理方法是，打印异常的跟踪栈信息，并中止程序运行，这就是程序在遇到异常后自动结束的原因。

throws声明抛出只能在方法签名中使用，throws可以声明抛出多个异常类，多个异常类之间以逗号隔开。

### 多catch语句
可以使用多个catch语句，每个catch分别捕获对应的Exception及其子类。JVM在捕获到异常后，会从上到下匹配catch语句，匹配到某个catch后，执行catch代码块，然后不再继续匹配。简单地说就是：多个catch语句只有一个能被执行。因此存在多个catch的时候，catch的顺序非常重要：子类必须写在前面。

### finally语句
Java的`try...catch`机制还提供了`finally`语句，finally语句块保证有无错误都会执行。

注意finally有几个特点：
* finally语句不是必须的，可写可不写；
* finally总是最后执行。

如果没有发生异常，就正常执行try { ... }语句块，然后执行finally。如果发生了异常，就中断执行try { ... }语句块，然后跳转执行匹配的catch语句块，最后执行finally。在catch中抛出异常，不会影响finally的执行。JVM会先执行finally，然后抛出异常。

某些情况下，可以没有catch，只使用try ... finally结构。

### 多异常捕获
如果某些异常的处理逻辑相同，但是异常本身不存在继承关系，可以使用一个catch块可以捕获多种类型的异常：
* 捕获多种类型的异常时，多种异常类型之间用竖线（|）隔开。
* 捕获多种类型的异常时，异常变量有隐式的final修饰，因此程序不能对异常变量重新赋值。

### 访问异常信息
所有的异常对象都包含了如下几个常用方法：
* getMessage()：返回该异常的详细描述字符串。
* printStackTrace()：将该异常的跟踪栈信息输出到标准错误输出。
* printStackTrace(PrintStream s)：将该异常的跟踪栈信息输出到指定输出流。
* getStackTrace()：返回该异常的跟踪栈信息。

## 自动关闭资源的try语句
在try关键字后紧跟一对圆括号，圆括号可以声明、初始化一个或多个资源（使用分号分割多个资源），此处的资源指的是那些必须在程序结束时显式关闭的资源（比如数据库连接、网络连接等），try语句在该语句结束时自动关闭这些资源。

需要指出的是，为了保证try语句可以正常关闭资源，这些资源实现类必须实现AutoCloseable或Closeable(`Closeable extends AutoCloseable`)接口，实现这两个接口就必须实现close()方法。

## 异常跟踪栈
只要异常没有被完全捕获（包括异常没有被捕获，或异常被处理后重新抛出了新异常），异常从发生异常的方法逐渐向外传播，首先传给该方法的调用者，该方法调用者再次传给其调用者……直至最后传到main方法，如果main方法依然没有处理该异常，JVM会中止该程序，并打印异常的跟踪栈信息。

运行下面的程序：
```java
public class T {
    private void a() {
        b();
    }

    private void b() {
        c();
    }

    private void c() {
        System.out.println(1 / 0);
    }

    public static void main(String[] args) {
        new T().a();
    }
}
```
输出的结果如下：
```
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at fun.pigyun.test.T.c(T.java:13)
	at fun.pigyun.test.T.b(T.java:9)
	at fun.pigyun.test.T.a(T.java:5)
	at fun.pigyun.test.T.main(T.java:17)

Process finished with exit code 1
```
第一行的信息详细显示了异常的类型和异常的详细消息。接下来跟踪栈记录程序中所有的异常发生点，各行显示被调用方法中执行的停止位置，并标明类、类中的方法名、与故障点对应的文件的行。一行行地往下看，跟踪栈总是最内部的被调用方法逐渐上传，直到最外部业务操作的起点，通常就是程序的入口main方法或Thread类的run方法（多线程的情形）。


## 异常屏蔽
如果在执行finally语句时抛出异常，那么原来在catch中准备抛出的异常就“消失”了，因为只能抛出一个异常。没有被抛出的异常称为“被屏蔽”的异常（Suppressed Exception）。

正确的方法是先用origin变量保存原始异常，然后调用Throwable.addSuppressed()，把原始异常添加进来，最后在finally抛出：
```Java
Exception origin = null;
try {
    System.out.println(Integer.parseInt("abc"));
} catch (Exception e) {
    origin = e;
    throw e;
} finally {
    Exception e = new IllegalArgumentException();
    if (origin != null) {
        e.addSuppressed(origin);
    }
    throw e;
}
```
当catch和finally都抛出了异常时，虽然catch的异常被屏蔽了，但是，finally抛出的异常仍然包含了它：
```
Exception in thread "main" java.lang.IllegalArgumentException
    at Main.main(Main.java:11)
Suppressed: java.lang.NumberFormatException: For input string: "abc"
    at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.base/java.lang.Integer.parseInt(Integer.java:652)
    at java.base/java.lang.Integer.parseInt(Integer.java:770)
    at Main.main(Main.java:6)
```

通过Throwable.getSuppressed()可以获取所有的Suppressed Exception。

绝大多数情况下，在finally中不要抛出异常。因此，我们通常不需要关心Suppressed Exception。


## NullPointerException
NullPointerException即空指针异常，俗称NPE。如果一个对象为null，调用其方法或访问其字段就会产生NullPointerException，这个异常通常是由JVM抛出的。

指针(Pointer)这个概念实际上源自C语言，Java语言中并无指针。我们定义的变量实际上是引用，Null Pointer更确切地说是Null Reference，不过两者区别不大。

如果产生了NullPointerException，例如，调用a.b.c.x()时产生了NullPointerException，原因可能是：
* a是null
* a.b是null
* a.b.c是null

确定到底是哪个对象是null，Java14以前只能打印这样的日志：
```
System.out.println(a);
System.out.println(a.b);
System.out.println(a.b.c);
```

从Java 14开始，如果产生了NullPointerException，JVM可以给出详细的信息告诉我们null对象到底是谁。这种增强的NullPointerException详细信息是Java 14新增的功能，但默认是关闭的，我们可以给JVM添加一个-XX:+ShowCodeDetailsInExceptionMessages参数启用它：
```sh
java -XX:+ShowCodeDetailsInExceptionMessages Main.java
```