# NIO.2（New Input/Output 2）
NIO.2提供了全面的文件IO和文件系统访问支持，以及基于异步Channel的IO。

## Path、Paths和Files
* Path接口代表了一个平台无关的平台路径
* Files和Paths是两个工具类
* 其中Files包含了大量静态的工具方法来操作文件（复制、读取、是否隐藏、大小、写入……），还可以使用Stream API来操作文件目录和文件内容。
* Paths则包含了两个返回Path的静态工厂方法（get），其中`get(String first, String...more)`方法可以用来获取Path对象，Paths会将给定的多个字符串连缀成路径。

## 使用FileVisitor遍历文件和目录
Files类提供了如下两个方法来遍历文件和子目录：
* `walkFileTree(Path start, FileVisitor<?super Path>visitor)`：遍历start路径下的所有文件和子目录。
* `walkFileTree(Path start, Set<FileVisitOption> options, int maxDepth, FileVisitor<?super Path>visitor)`：与上一个方法的功能类似。该方法最多遍历maxDepth深度的文件。

上面2个方法都需要FileVisitor参数，FileVisitor代表一个文件访问器，walkFileTree()方法会自动遍历start路径下的所有文件和子目录，遍历文件和子目录都会“触发”FileVisitor中相应的方法。FileVisitor中定义了如下4个方法：
1. FileVisitResult postVisitDirectory(T dir, IOException exc)：访问子目录之后触发该方法。
2. FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs)：访问子目录之前触发该方法。
3. FileVisitResult visitFile(T file, BasicFileAttributes attrs)：访问file文件时触发该方法。
4. FileVisitResult visitFileFailed(T file, IOException exc)：访问file文件失败时触发该方法。

上面4个方法都返回一个FileVisitResult对象，它是一个枚举类，代表了访问之后的后续行为。FileVisitResult定义了如下几种后续行为：
* CONTINUE：代表“继续访问”的后续行为。
* SKIP_SIBLINGS：代表“继续访问”的后续行为，但不访问该文件或目录的兄弟文件或目录。
* SKIP_SUBTREE：代表“继续访问”的后续行为，但不访问该文件或目录的子目录树。
* TERMINATE：代表“中止访问”的后续行为。

实际编程时没必要为FileVisitor的4个方法都提供实现，可以通过继承SimpleFileVisitor（FileVisitor的实现类）来实现自己的“文件访问器”，这样就根据需要、选择性地重写指定方法了。

## 使用WatchService监控文件变化
NIO.2的Path类提供了如下一个方法来监听文件系统的变化：  
`register(WatchService watcher, WatchEvent.Kind<?>...events)`：用watcher监听该path代表的目录下的文件变化。events参数指定要监听哪些类型的事件。

在这个方法中WatchService代表一个文件系统监听服务，它负责监听path代表的目录下的文件变化。一旦使用register()方法完成注册之后，接下来就可调用WatchService的如下三个方法来获取被监听目录的文件变化事件：
* WatchKey poll()：获取下一个WatchKey，如果没有WatchKey发生就立即返回null。
* WatchKey poll(long timeout, TimeUnit unit)：尝试等待timeout时间去获取下一个WatchKey。
* WatchKey take()：获取下一个WatchKey，如果没有WatchKey发生就一直等待。

如果程序需要一直监控，则应该选择使用take()方法；如果程序只需要监控指定时间，则可考虑使用poll()方法。

## 访问文件属性
的NIO.2在java.nio.file.attribute包下提供了大量的工具类，通过这些工具类，开发者可以非常简单地读取、修改文件属性。这些工具类主要分为如下两类：
* XxxAttributeView：代表某种文件属性的“视图”。
* XxxAttributes：代表某种文件属性的“集合”，程序一般通过XxxAttributeView对象来获取XxxAttributes。

FileAttributeView是其他XxxAttributeView的父接口，常用的XxxAttributeView如下：
* AclFileAttributeView：通过AclFileAttributeView，开发者可以为特定文件设置ACL（Access Control List）及文件所有者属性。它的getAcl()方法返回List<AclEntry>对象，该返回值代表了该文件的权限集。通过setAcl（List）方法可以修改该文件的ACL。
* BasicFileAttributeView：它可以获取或修改文件的基本属性，包括文件的最后修改时间、最后访问时间、创建时间、大小、是否为目录、是否为符号链接等。它的readAttributes()方法返回一个“BasicFileAttributes对象，对文件夹基本属性的修改是通过BasicFileAttributes对象完成的。
* DosFileAttributeView：它主要用于获取或修改文件DOS相关属性，比如文件是否只读、是否隐藏、是否为系统文件、是否是存档文件等。它的readAttributes()方法返回一个DosFileAttributes对象，对这些属性的修改其实是由DosFileAttributes对象来完成的。
* FileOwnerAttributeView：它主要用于获取或修改文件的所有者。它的getOwner()方法返回一个UserPrincipal对象来代表文件所有者；也可调用setOwner（UserPrincipal owner）方法来改变文件的所有者。
* PosixFileAttributeView：它主要用于获取或修改POSIX（Portable Operating System Interface of INIX）属性，它的readAttributes()方法返回一个PosixFileAttributes对象，该对象可用于获取或修改文件的所有者、组所有者、访问权限信息（就是UNIX的chmod命令负责干的事情）。这个View只在UNIX、Linux等系统上有用。
* UserDefinedFileAttributeView：它可以让开发者为文件设置一些自定义属性。