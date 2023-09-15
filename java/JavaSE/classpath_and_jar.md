# classpath 和 jar包



## classpath

### 含义
classpath是JVM用到的一个环境变量，它用来指示JVM如何搜索class。

因为Java是编译型语言，源码文件是.java，而编译后的.class文件才是真正可以被JVM执行的字节码。因此，JVM需要知道，如果要加载一个abc.xyz.Hello的类，应该去哪搜索对应的Hello.class文件。

所以，classpath就是一组目录的集合，它设置的搜索路径与操作系统相关：
* 在Windows系统上，用`;`分隔
* 在Linux系统上，用``:`分隔
* 带空格的目录用`""`括起来

当JVM在加载abc.xyz.Hello这个类时，会依次查找这个目录的集合。如果JVM在某个路径下找到了对应的class文件，就不再往后继续搜索。如果所有路径下都没有找到，就报错。

### 设定
classpath的设定方法有两种：
* 在系统环境变量中设置classpath环境变量，不推荐
* 在启动JVM时设置classpath变量，推荐

在启动JVM时设置classpath实际上就是给java命令传入-classpath或-cp参数。如果没有设置系统环境变量，也没有传入-cp参数，那么JVM默认的classpath为`.`，即当前目录。

### 示例
假设我们有一个编译后的Hello.class，它的包名是abc.xyz，当前目录是C:\work，那么，目录结构必须如下：
```
C:\work
└─ abc
   └─ xyz
      └─ Hello.class
```
运行这个Hello.class必须在当前目录下使用如下命令：`C:\work> java -cp . abc.xyz.Hello`

JVM根据classpath设置的.在当前目录下查找abc.xyz.Hello，即实际搜索文件必须位于abc/xyz/Hello.class。如果指定的.class文件不存在，或者目录结构和包名对不上，均会报错。



## jar包

### 作用
jar包可以把package组织的目录层级，以及各个目录下的所有文件（包括.class文件和其他文件）都打成一个jar文件，方便管理。

### 本质
jar包就是zip包，制作jar包就是制作一个zip文件，把后缀从.zip改为.jar，一个jar包就创建成功。

![jar+20230122231044](https://raw.githubusercontent.com/loli0con/picgo/master/images/jar%2B20230122231044.png%2B2023-01-22-23-10-45)

### 执行
如果我们要执行一个jar包的class，就可以把jar包放到classpath中：`java -cp ./hello.jar abc.xyz.Hello`。这样JVM会自动在hello.jar文件里去搜索某个类。

### MANIFEST
jar包还可以包含一个特殊的/META-INF/MANIFEST.MF文件，MANIFEST.MF是纯文本，可以指定Main-Class和其它信息。JVM会自动读取这个MANIFEST.MF文件，如果存在Main-Class，我们就不必在命令行指定启动的类名，而是用更方便的命令：`java -jar hello.jar`。jar包还可以包含其它jar包，这个时候，就需要在MANIFEST.MF文件里配置classpath了。

在大型项目中，不可能手动编写MANIFEST.MF文件，再手动创建zip包。Java社区提供了大量的开源构建工具，例如Maven，可以非常方便地创建jar包。