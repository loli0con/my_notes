# 包

## 基本概念
Java定义了一种名字空间，称之为包：**package**。一个类总是属于某个包，类名（比如`String`）只是一个简写，真正的完整类名是包名.类名（`java.lang.String`）。

包可以是多层结构，用`.`隔开。例如：`java.util`。包没有父子关系，`java.util`和`java.util.zip`是不同的包，两者没有任何继承关系。

在Java虚拟机执行的时候，JVM只看完整类名，因此，只要包名不同，类就不同。

Java包机制需要两个方面的保证
1. 源文件里使用package语句指定包名
2. class文件必须放在对应的路径下

同一个包中的类不必位于相同的目录下，只要让CLASSPATH环境变量里包含这两个路径即可。jvm会自动搜索CLASSPATH下的子路径，把它们当成用一个包下的类来处理。

## package关键字
没有定义包名的**class**，它使用的是默认包，非常容易引起名字冲突，因此，不推荐不写包名的做法。

在定义**class**的时候，需要在第一行声明这个**class**属于哪个包。

```Java
package ming; // 申明包名ming

public class Person {
}
```

```Java
package mr.jun; // 申明包名mr.jun

public class Arrays {
}
```

还需要按照包结构把上面的Java文件组织起来。假设以package_sample作为根目录，src作为源码目录，那么所有文件结构就是：
```
package_sample
└─ src
    ├─ hong
    │  └─ Person.java
    │  ming
    │  └─ Person.java
    └─ mr
       └─ jun
          └─ Arrays.java
```
即所有Java文件对应的目录层次要和包的层次一致。

编译后的.class文件也需要按照包结构存放。如果使用IDE，把编译后的.class文件放到bin目录下，那么，编译的文件结构就是：
```
package_sample
└─ bin
   ├─ hong
   │  └─ Person.class
   │  ming
   │  └─ Person.class
   └─ mr
      └─ jun
         └─ Arrays.class
```


## 作用域
位于同一个包的类，可以访问包作用域的字段和方法。不用public、protected、private修饰的字段和方法就是包作用域。


## import关键字
### import
import可以向某个Java文件中导入指定包层次下某个类或者全部类：
* import package.subpackage...Classname
* import package.subpackage...*

### import static 静态导入（不推荐使用）
import static用于导入指定类的某个静态成员变量、方法或全部的静态成员变量、方法：
* import static package.subpackage...ClassName.fieldName|methodName
* import static package.subpackage...ClassName.*


## 编译过程
Java编译器最终编译出的.class文件只使用完整类名，因此，在代码中，当编译器遇到一个class名称时：
1. 如果是完整类名，就直接根据完整类名查找这个class
2. 如果是简单类名，按下面的顺序依次查找：
   1. (默认自动import当前package的其他class)查找当前package是否存在这个class
   2. 查找import的包是否包含这个class
   3. (默认自动import java.lang.*)查找java.lang包是否包含这个class
3. 如果按照上面的规则还无法确定类名，则编译报错。


## 编译示例
假设我们创建了如下的目录结构：
```
work
├── bin
└── src
    └── com
        └── itranswarp
            ├── sample
            │   └── Main.java
            └── world
                └── Person.java
```
其中，bin目录用于存放编译后的class文件，src目录按包结构存放Java源码。

首先，确保当前目录是work目录，即存放src和bin的父目录：
```sh
$ ls
bin src
```
然后，编译src目录下的所有Java文件：
```sh
javac -d ./bin src/**/*.java
```
命令行-d指定输出的class文件存放bin目录，后面的参数src/**/*.java表示src目录下的所有.java文件，包括任意深度的子目录。

注意：Windows不支持**这种搜索全部子目录的做法，所以在Windows下编译必须依次列出所有.java文件：
```sh
C:\work> javac -d bin src\com\itranswarp\sample\Main.java src\com\itranswarp\world\Persion.java
```

如果编译无误，则javac命令没有任何输出。可以在bin目录下看到如下class文件：
```
bin
└── com
    └── itranswarp
        ├── sample
        │   └── Main.class
        └── world
            └── Person.class
```

现在，我们就可以直接运行class文件了。根据当前目录的位置确定classpath，例如，当前目录仍为work，则classpath为bin或者./bin：
```sh
$ java -cp bin com.itranswarp.sample.Main 
Hello, world!
```


## 最佳实践
为了避免名字冲突，我们需要确定唯一的包名。推荐的做法是**使用倒置的域名来确保唯一性**。例如：
* org.apache
* org.apache.commons.log
* com.liaoxuefeng.sample

**子包就可以根据功能自行命名**。

要注意**不要和java.lang包的类重名**，即自己的类不要使用这些名字：
* String
* System
* Runtime
* ...

要注意也**不要和JDK常用类重名**：
* java.util.List
* java.text.Format
* java.math.BigInteger
* ...
