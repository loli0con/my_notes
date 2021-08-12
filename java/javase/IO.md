# 输入/输出
这里的IO主要是关于文件的，网络IO在另外的笔记里面。

## File类
File类代表与平台无关的文件和目录，通过File类可以在程序中操作文件和目录。

### 创建实例
File类可以使用文件路径字符串来创建File实例，该文件路径字符串既可以是绝对路径，也可以是相对路径。

### 访问文件和目录
![IO+20210612091759](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612091759.png%2B2021-06-12-09-18-01)
![IO+20210612091822](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612091822.png%2B2021-06-12-09-18-24)

### 文件过滤器
在File类的list()方法中可以接受一个FilenameFilter参数，通过该参数可以只列出符合条件的文件。  
FilenameFilter接口里包含一个accept(File dir, String name)方法，该方法将以此对指定File的所有子目录或者文件进行迭代，如果该方法放回true，则list()方法会列出该子目录或文件。

## IO流
在Java中把不同的输入/输出源（键盘、文件、网络连接等）抽象表述成“流（stream）”，通过流的方式允许Java程序使用相同的方式来访问不同的输入/输出流。  
stream是从起源（source）到接收（sink）的有序数据。

### 流的分类

#### 输入流和输出流
按照流的流向来分，可以分为输入流和输出流：
* 输入流：只能从中读取数据，而不能向其写入数据。
* 输出流：只能向其写入数据，而不能从中读取数据。

这里的输入、输出都是从程序运行所在内存的角度来划分的。

Java的输入流主要由InputStream和Reader作为基类，而输出流主要由OutputStream和Writer作为基类。它们都是一些抽象类，无法直接创建实例。

#### 字节流和字符流
字节流操作的数据单元是8位的字节，而字符流操作的数据单元是16位的字符。

字节流主要由InputStream和OutputStream作为基类，而字符流则主要由Reader和Writer作为基类。

#### 节点流和处理流
按照流的角色来分，可以分为节点流和处理流：
* 可以从/向一个特定的IO设备（如磁盘、网络）读/写的数据的流，成为节点流，节点流也被称为低级流。
* 处理流则对一个已存在的流进行连接和封装，通过封装后的流来实现数据读/写功能，处理流也被称为高级流。

![IO+20210612093132](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612093132.png%2B2021-06-12-09-31-33)

Java使用处理流来包装节点流是一种典型的装饰器设计模式，通过使用处理流来包装不同节点流，既可以消除不同节点流的实现差异，也可以提供更方便的方法来完成输入/输出功能，因此处理流也被称为包装流。

### 输入流
![IO+20210612102422](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612102422.png%2B2021-06-12-10-24-24)

如果read方法返回-1，即表明到了输入流的结束点。

程序里打开的文件IO资源不属于内存里的资源，垃圾回收机制无法回收该资源，所以应该显式关闭文件IO资源。所有的IO资源类，它们都实现了AutoCloseable接口，因此都可以通过自动关闭资源的try语句来关闭这些IO流。

![IO+20210612102721](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612102721.png%2B2021-06-12-10-27-21)

### 输出流
![IO+20210612102754](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612102754.png%2B2021-06-12-10-27-55)

关闭数据流可以将缓冲区中的数据flush到物理节点里，因为在执行close()方法之间，自动执行了数据流的flush()方法。

### 处理流
使用处理流时的典型思路是，使用处理流来包装节点流，程序通过处理流来执行输入/输出功能，让节点流与底层的I/O设备、文件交互。  
例如`new PrintStram(new FileOutputStream("文件名"))`。

关闭最上层的处理流时，系统会自动关闭被该处理流包装的节点流。

### 输入/输出流体系
![IO+20210612103420](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612103420.png%2B2021-06-12-10-34-22)

### 重定向标准输入/输出
Java的标准输入/输出分别通过System.in和System.out来代表，在默认情况下它们分别代表键盘和显示器。  
在System类里提供了如下三个重定向标准输入/输出的方法：
* static void setErr(PrintStream err)：重定向“标准”错误输出流
* static void setIn(InputStream in)：重定向“标准”输入流
* static void setOut(PrintStream out)：重定向“标准”输出流

### Java虚拟机读写其他进程的数据
使用Runtime对象的exec()方法可以运行平台上的其他程序，该方法产生一个Process对象，Process对象代表由该Java程序启动的子进程。  
Process类提供了如下三个方法，用于让程序和其子程序进行通信：
* InputStream getErrorStream()：获得子进程的错误流。
* InputStream getInputStream()：获得子进程的输入流。
* OutputStream getOutputStream()：获得子进程的输出流。

![IO+进程的输入和输出](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%92%8C%E8%BE%93%E5%87%BA.png%2B2021-06-12-11-08-21)

### RandomAccessFile
RandomAccessFile是Java输入/输出流体系中功能最丰富的文件内容访问类，它提供了众多的方法来访问文件内容，它既可以读取文件内容，也可以向文件输出数据，支持“随机访问”的方式——可以直接跳到文件的任意位置来读写数据。

RandomAccessFile可以自由移动文件记录指针，可以向前移动，也可以向后移动，它包含了如下两个方法来操作文件记录指针：
* long getFilePointer()：返回文件记录指针的当前位置。
* void seek(long pos)：将文件记录指针定位到pos位置。

![IO+20210612111800](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612111800.png%2B2021-06-12-11-18-01)

RandomAccessFile类有两个构造器，一个使用String参数来指定文件名，一个使用File参数来指定文件本身。除此之外，创建RandomAccessFile对象时，还需要指定一个mode参数，该参数指定RandomAccessFile的访问模式，该参数有如下4个值：
* "r"：以只读的方式打开指定文件。如果试图对该RandomAccessFile执行写入方法，都将抛出IOException异常。
* "rw"：以读、写方式打开指定文件。如果该文件尚不存在，则尝试创建该文件。
* "rwd"：以读、写方式打开指定文件。相对于"rw"模式，还要求对文件的内容的每个更新都同步写入到底层存储设备。
* "rws"：以读、写方式打开指定文件。相对于"rw"模式，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备。

## 序列化

### 基本概念
序列化是把内存中的对象转换成二进制内容，从而进行存储或者传输。  
Java对象 --> byte[]

反序列化，即把一个二进制内容变回对象。  
byte[] --> Java对象

### 可序列化
如果要让某个对象支持序列化机制，则它的类必须是可序列化的。为了让某个类是可序列化的，则该类必须实现如下两个接口之一：
* Serializable：  
Serializable是一个“标记接口”——它是一个空接口，没有定义任何方法。
* Externalizable


### 序列化
一旦某个类实现了Serializable接口，该类的对象就是可序列化的，程序可通过如下两个步骤序列化该对象：
1. 创建一个ObjectOutputStream，这个输出流是一个处理流，必须建立在其他节点流的基础之上。
2. 调用ObjectOutputStream对象的writeObject()方法输出可序列化对象。

### 反序列化
如果希望从二进制的内容中恢复对象，则需要使用反序列化，步骤如下：
1. 创建一个ObjectInputStream输入流，这个输入流是一个处理流，必须建立在其他节点流的基础之上。
2. 调用ObjectInputStream对象的readObject()方法读取流中的对象，该方法返回一个Object类型的Java对象。如果知道该Java对象的类型，则可以将该对象强制类型转换成其真实的类型。

必须指出的是，反序列化读取的仅仅是Java对象的数据，而不是Java类，因此采用反序列化恢复Java对象时，必须提供该Java对象所属类的class文件，否则将会引发ClassNotFoundException异常。

反序列化无需通过构造器来初始化Java对象。

当一个可序列化类有多个父类时（包括直接父类和间接父类），这些父类要么有无参数的构造器，要么也是可序列化的——否则反序列化时将抛出InvalidClassException异常。如果父类是不可序列化的，只是带有无参数的构造器，则该父类中定义的成员变量值不会序列化到二进制流中。

### 对象引用
如果某个类的成员变量的类型不是基本类型或String类型，而是另一个引用类型，那么这个引用类型必须是可序列化的，否则拥有该引用类型成员变量的类也是不可序列化的，无论是否实现Serializable、Externalizable接口。

Java序列化机制采用了一种特殊的序列化算法：
* 所有保存到磁盘中的对象都有一个序列化编号
* 当程序试图序列化一个对象时，程序将先检查该对象是否已经被序列化过，只有该对象从未（在本次虚拟机中）被序列化过，系统才会将该对象转换成字节序列并输出。
* 如果某个对象已经被序列化过，程序将只是直接输出一个序列化编号，而不是再次重新序列化该对象。

![IO+20210621010430](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621010430.png%2B2021-06-21-01-04-31)

![IO+20210621010746](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621010746.png%2B2021-06-21-01-07-47)

序列化前后对象的地址不同了，但是内容是一样的，而且对象中包含的引用也相同。换句话说，通过序列化操作,我们可以实现对任何可Serializable对象的“深度复制（deep copy）”。
### 自定义序列化
通过在实例变量前面使用transient关键字修饰，被transient修饰的实例变量将被完全隔离在序列化机制之外。

Java还提供了一种自定义序列化机制，通过这种自定义序列化机制，可以让程序控制如何实例化各实例变量。在序列化和反序列化过程中需要特殊处理的类应提供如下特殊签名的方法，这些特殊的方法用以实现自定义序列化：
* private void writeObject(java.io.ObjectOutputStream out)throws IOException;
* private void readObject(java.io.ObjectInputStream in)throws IOException, ClassNotFoundException;
* private void readObjectNoData()throws ObjectStreamException;

![IO+20210621091722](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621091722.png%2B2021-06-21-09-17-23)

![IO+20210621091953](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621091953.png%2B2021-06-21-09-19-53)

![IO+20210621092421](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621092421.png%2B2021-06-21-09-24-21)

![IO+20210621092522](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621092522.png%2B2021-06-21-09-25-23)

![IO+20210621092554](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621092554.png%2B2021-06-21-09-25-54)

### Externalizable —— 另一种自定义序列化机制
![IO+20210621092726](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621092726.png%2B2021-06-21-09-27-27)

![IO+20210621092926](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621092926.png%2B2021-06-21-09-29-26)

### 版本
![IO+20210621093142](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621093142.png%2B2021-06-21-09-31-42)

![IO+20210621093201](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210621093201.png%2B2021-06-21-09-32-03)

## NIO
新IO采用内存内存映射文件的方式来处理输入/输出，新IO将文件或文件的一段区域映射到内存中，像访问内存一样来访问文件（模拟了虚拟内存的概念）。新IO是面向块/缓存的。

Channel（通道）和Buffer（缓冲）是新IO中的两个核心对象。  
Channel是对传统的输入/输出系统的模拟，在新IO系统中所有的数据都需要通过通道传输。Buffer可以被理解成一个容器，它的本质是一个数组，发送到Channel中的所有对象都必须首先放到Buffer中，而从Channel中读取的数据也必须先放到Buffer中。

### Buffer
Buffer是一个抽象类，其最常用的子类是ByteBuffer，它可以在底层字节数组上进行get/set操作。除了ByteBuffer之外，对应于其他基本数据类型（boolean除外）都有相应的Buffer类：CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。

![IO+20210612193841](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612193841.png%2B2021-06-12-19-38-43)

![IO+20210612193952](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612193952.png%2B2021-06-12-19-39-53)

![IO+20210612194034](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612194034.png%2B2021-06-12-19-40-35)

通过allocate()方法创建的Buffer对象都是普通Buffer，ByteBuffer还提供了一个allocateDirect()方法来创建直接Buffer。直接Buffer的创建成本比普通Buffer的创建成本高，但直接Buffer的读取效率更高。

### Channel
Channel类似于传统的流对象，但是有两个主要的区别：
* Channel可以直接将指定文件的部分活着全部直接映射成Buffer。
* 程序不能直接访问Channel中的数据，包括读写都不行，Channel只能与Buffer进行交互。

Channel通过传统的节点InputStream、OutputStream的getChannel()方法来获得，不同的节点流获得的Channel不一样。

Channel中最常用的三类方法是map()、read()和write()。  
map()方法用于将Channel对应的部分或者全部数据映射成ByteBuffer；read()或write()方法用于从Buffer中读取或向Buffer中写入数据。

### Charset
Java默认使用Unicode字符集，Java使用Charset来处理字节序列和字符序列（字符串）之间的转换关系，该类包含了用于创建解码器和编码器的方法，还提供了获取Charset所支持字符集的方法，Charset类是不可变的。

![IO+20210612201043](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612201043.png%2B2021-06-12-20-10-44)

### 文件锁
![IO+20210612201254](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612201254.png%2B2021-06-12-20-12-55)

## NIO.2
NIO.2提供了全面的文件IO和文件系统访问支持，以及基于异步Channel的IO。

### Path、Paths和Files
Path接口代表了一个平台无关的平台路径。  
Files和Paths是两个工具类，其中Files包含了大量静态的工具方法来操作文件（复制、读取、是否隐藏、大小、写入……）；Paths则包含了两个返回Path的静态工厂方法（get）。

还可以使用Stream API来操作文件目录和文件内容。

#### FileVisitor
![IO+Files类提供了如下两个方法来遍历文件和子目录](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2BFiles%E7%B1%BB%E6%8F%90%E4%BE%9B%E4%BA%86%E5%A6%82%E4%B8%8B%E4%B8%A4%E4%B8%AA%E6%96%B9%E6%B3%95%E6%9D%A5%E9%81%8D%E5%8E%86%E6%96%87%E4%BB%B6%E5%92%8C%E5%AD%90%E7%9B%AE%E5%BD%95%EF%BC%9A%0A.png%2B2021-06-12-20-36-40)

#### WatchService
![IO+Path类提供了一个方法来监听文件系统的变化](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2BPath%E7%B1%BB%E6%8F%90%E4%BE%9B%E4%BA%86%E4%B8%80%E4%B8%AA%E6%96%B9%E6%B3%95%E6%9D%A5%E7%9B%91%E5%90%AC%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%8F%98%E5%8C%96%EF%BC%9A.png%2B2021-06-12-20-56-24)

[WatchKey——监控池的介绍](https://blog.csdn.net/Lirx_Tech/article/details/51425364)

### 更多的文件属性
![IO+20210612210537](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612210537.png%2B2021-06-12-21-05-39)

## 后记
http://zhongmingmao.me/2019/07/22/java-performance-io-model/  
http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html