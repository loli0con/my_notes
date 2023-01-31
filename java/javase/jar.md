# jar包

## 作用
jar包可以把package组织的目录层级，以及各个目录下的所有文件（包括.class文件和其他文件）都打成一个jar文件，方便管理。

## 本质
jar包就是zip包，制作jar包就是制作一个zip文件，把后缀从.zip改为.jar，一个jar包就创建成功。

![jar+20230122231044](https://raw.githubusercontent.com/loli0con/picgo/master/images/jar%2B20230122231044.png%2B2023-01-22-23-10-45)


## MANIFEST
jar包还可以包含一个特殊的/META-INF/MANIFEST.MF文件，MANIFEST.MF是纯文本，可以指定Main-Class和其它信息。JVM会自动读取这个MANIFEST.MF文件，如果存在Main-Class，我们就不必在命令行指定启动的类名，而是用更方便的命令：`java -jar hello.jar`。jar包还可以包含其它jar包，这个时候，就需要在MANIFEST.MF文件里配置classpath了。

## 构建
在大型项目中，不可能手动编写MANIFEST.MF文件，再手动创建zip包。Java社区提供了大量的开源构建工具，例如Maven，可以非常方便地创建jar包。