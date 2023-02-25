# 流
在Java中把不同的输入/输出源（键盘、文件、网络连接等）抽象表述成“流（stream）”，通过流的方式允许Java程序使用相同的方式来访问不同的输入/输出流。stream是从起源（source）到接收（sink）的有序数据。


## 流的分类

### 输入流和输出流
按照流的流向来分，可以分为输入流和输出流：
* 输入流：只能从中读取数据，而不能向其写入数据。
* 输出流：只能向其写入数据，而不能从中读取数据。

这里的输入、输出都是从程序运行所在内存的角度来划分的。

Java的输入流主要由InputStream和Reader作为基类，而输出流主要由OutputStream和Writer作为基类。它们都是一些抽象类，无法直接创建实例。

### 字节流和字符流
字节流操作的数据单元是8位的Byte字节，而字符流操作的数据单元是16位的Char字符。

字节流主要由InputStream和OutputStream作为基类，而字符流则主要由Reader和Writer作为基类。

### 节点流和处理流
按照流的角色来分，可以分为节点流和处理流：
* 可以从/向一个特定的IO设备（如磁盘、网络）读/写的数据的流，成为节点流，节点流也被称为低级流。
* 处理流则对一个已存在的流进行连接和封装，通过封装后的流来实现数据读/写功能，处理流也被称为高级流。

![IO+20210612093132](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612093132.png%2B2021-06-12-09-31-33)

Java使用处理流来包装节点流是一种典型的装饰器设计模式，通过使用处理流来包装不同节点流，既可以消除不同节点流的实现差异，也可以提供更方便的方法来完成输入/输出功能，因此处理流也被称为包装流。


## 流的方法

### 输入流
InputStream和Reader是所有输入流的抽象基类，本身并不能创建实例来执行输入，但它们将成为所有输入流的模板，所以它们的方法是所有输入流都可使用的方法。

在InputStream里包含如下三个方法：
1. int read()：从输入流中读取单个字节，返回所读取的字节数据（字节数据可直接转换为int类型）。
2. int read(byte[] b)：从输入流中最多读取b.length个字节的数据，并将其存储在字节数组b中，返回实际读取的字节数。
3. int read(byte[] b, int off, int len)：从输入流中最多读取len个字节的数据，并将其存储在数组b中，放入数组b中时，并不是从数组起点开始，而是从off位置开始，返回实际读取的字节数。

在Reader里包含如下三个方法：
1. int read()：从输入流中读取单个字符，返回所读取的字符数据（字符数据可直接转换为int类型）。
2. int read(char[] cbuf)：从输入流中最多读取cbuf.length个字符的数据，并将其存储在字符数组cbuf中，返回实际读取的字符数。
3. int read(char[] cbuf, int off, int len)：从输入流中最多读取len个字符的数据，并将其存储在字符数组cbuf中，放入数组cbuf中时，并不是从数组起点开始，而是从off位置开始，返回实际读取的字符数。

如果read方法返回-1，即表明到了输入流的结束点。

程序里打开的文件IO资源不属于内存里的资源，垃圾回收机制无法回收该资源，所以应该显式关闭文件IO资源。所有的IO资源类，它们都实现了AutoCloseable接口，因此都可以通过自动关闭资源的try语句来关闭这些IO流。

InputStream和Reader还支持如下几个方法来移动记录指针。
* void mark(int readAheadLimit)：在记录指针当前位置记录一个标记（mark）。
* boolean markSupported()：判断此输入流是否支持mark()操作，即是否支持记录标记。
* void reset()：将此流的记录指针重新定位到上一次记录标记（mark）的位置。
* long skip(long n)：记录指针向前移动n个字节/字符。

### 输出流
OutputStream和Writer两个流都提供了如下三个方法：
1. void write(int c)：将指定的字节/字符输出到输出流中，其中c既可以代表字节，也可以代表字符。
2. void write(byte\[]/char[] buf)：将字节数组/字符数组中的数据输出到指定输出流中。
3. void write(byte\[]/char[] buf, int off, int len)：将字节数组/字符数组中从off位置开始，长度为len的字节/字符输出到输出流中。

因为字符流直接以字符作为操作单位，所以Writer可以用字符串来代替字符数组，即以String对象作为参数：
* void write(String str)：将str字符串里包含的字符输出到指定输出流中。
* void write(String str, int off, int len)：将str字符串里从off位置开始，长度为len的字符输出到指定输出流中。

OutputStream还提供了一个flush()方法，它的目的是将缓冲区的内容真正输出到目的地。通常情况下，不需要调用这个flush()方法，因为缓冲区写满了OutputStream会自动调用它。并且，在调用close()方法关闭OutputStream之前，也会自动调用flush()方法。

### 处理流
使用处理流时的典型思路是，使用处理流来包装节点流，程序通过处理流来执行输入/输出功能，让节点流与底层的I/O设备、文件交互。关闭最上层的处理流时，系统会自动关闭被该处理流包装的节点流。

```Java
// 确定数据源
InputStream file = new FileInputStream("test.gz");
// 提供缓冲功能
InputStream buffered = new BufferedInputStream(file);
// 读取解压缩的内容
InputStream gzip = new GZIPInputStream(buffered);
```
无论包装多少次，得到的对象始终是InputStream，直接用InputStream来引用它，就可以正常读取：
```
┌─────────────────────────┐
│GZIPInputStream          │
│┌───────────────────────┐│
││BufferedFileInputStream││
││┌─────────────────────┐││
│││   FileInputStream   │││
││└─────────────────────┘││
│└───────────────────────┘│
└─────────────────────────┘
```


## 流的缓冲
在读取流的时候，一次读取一个字节并不是最高效的方法。很多流支持一次性读取多个字节到缓冲区，对于文件和网络流来说，利用缓冲区一次性读取多个字节效率往往要高很多。InputStream提供了两个重载方法来支持读取多个字节：
* int read(byte[] b)：读取若干字节并填充到byte[]数组，返回读取的字节数
* int read(byte[] b, int off, int len)：指定byte[]数组的偏移量和最大填充数

利用上述方法一次读取多个字节时，需要先定义一个byte[]数组作为缓冲区，read()方法会尽可能多地读取字节到缓冲区， 但不会超过缓冲区的大小。read()方法的返回值不再是字节的int值，而是返回实际读取了多少个字节。如果返回-1，表示没有更多的数据了。


## 流的体系
![IO+20210612103420](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B20210612103420.png%2B2021-06-12-10-34-22)


### 管道流
表15.1中列出了4个访问管道的流：PipedInputStream、PipedOutputStream、PipedReader、PipedWriter，它们都是用于实现进程之间通信功能的，分别是字节输入流、字节输出流、字符输入流和字符输出流。

### 转换流
输入/输出流体系中还提供了两个转换流，这两个转换流用于实现将字节流转换成字符流，其中InputStreamReader将字节输入流转换成字符输入流，OutputStreamWriter将字节输出流转换成字符输出流。

普通的Reader实际上是基于InputStream构造的，因为Reader需要从InputStream中读入字节流（byte），然后根据编码设置，再转换为char就可以实现字符流。查看FileReader的源码，它在内部实际上持有一个FileInputStream。Reader本质上是一个基于InputStream的byte到char的转换器，如果已经有一个InputStream，想把它转换为Reader，可以使用InputStreamReader作为转换器，它可以把任何InputStream转换为Reader。示例代码如下：
```Java
// 持有InputStream:
InputStream input = new FileInputStream("src/readme.txt");
// 变换为Reader:
Reader reader = new InputStreamReader(input, "UTF-8");
```
构造InputStreamReader时，我们需要传入InputStream，还需要指定编码，就可以得到一个Reader对象。

### 推回输入流
PushbackInputStream和PushbackReader都提供了如下三个方法，这三个方法与InputStream和Reader中的三个read()方法一一对应：
* void unread(byte\[]/char[] buf)：将一个字节/字符数组内容推回到推回缓冲区里，从而允许重复读取刚刚读取的内容。
* void unread(byte\[]/char[] b, int off, int len)：将一个字节/字符数组里从off开始，长度为len字节/字符的内容推回到推回缓冲区里，从而允许重复读取刚刚读取的内容。
* void unread(int b)：将一个字节/字符推回到推回缓冲区里，从而允许重复读取刚刚读取的内容。

这两个推回输入流都带有一个推回缓冲区，当程序调用这两个推回输入流的unread()方法时，系统将会把指定数组的内容推回到该缓冲区里，而推回输入流每次调用read()方法时总是先从推回缓冲区读取，只有完全读取了推回缓冲区的内容后，但还没有装满read()所需的数组时才会从原输入流中读取。

当程序创建一个PushbackInputStream和PushbackReader时需要指定推回缓冲区的大小，默认的推回缓冲区的长度为1。如果程序中推回到推回缓冲区的内容超出了推回缓冲区的大小，将会引发Pushback buffer overflow的IOException异常。


### 重定向标准输入/输出
Java的标准输入/输出分别通过System.in和System.out来代表，在默认情况下它们分别代表键盘和显示器。在System类里提供了如下三个重定向标准输入/输出的方法：
* static void setErr(PrintStream err)：重定向“标准”错误输出流
* static void setIn(InputStream in)：重定向“标准”输入流
* static void setOut(PrintStream out)：重定向“标准”输出流

### Java虚拟机读写其他进程的数据
使用Runtime对象的exec()方法可以运行平台上的其他程序，该方法产生一个Process对象，Process对象代表由该Java程序启动的子进程。Process类提供了如下三个方法，用于让程序和其子程序进行通信：
* InputStream getErrorStream()：获得子进程的错误流。
* InputStream getInputStream()：获得子进程的输入流。
* OutputStream getOutputStream()：获得子进程的输出流。

从当前运行中的Java程序的视角去看：
![IO+进程的输入和输出](https://raw.githubusercontent.com/loli0con/picgo/master/images/IO%2B%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%92%8C%E8%BE%93%E5%87%BA.png%2B2021-06-12-11-08-21)

### RandomAccessFile
RandomAccessFile是Java输入/输出流体系中功能最丰富的文件内容访问类，它提供了众多的方法来访问文件内容，它既可以读取文件内容，也可以向文件输出数据，支持“随机访问”的方式——可以直接跳到文件的任意位置来读写数据。

#### 移动指针
RandomAccessFile对象也包含了一个记录指针，用以标识当前读写处的位置，当程序新创建一个RandomAccessFile对象时，该对象的文件记录指针位于文件头（也就是0处），当读/写了n个字节后，文件记录指针将会向后移动n个字节。除此之外，RandomAccessFile可以自由移动该记录指针，既可以向前移动，也可以向后移动。RandomAccessFile包含了如下两个方法来操作文件记录指针：
* long getFilePointer()：返回文件记录指针的当前位置。
* void seek(long pos)：将文件记录指针定位到pos位置。

#### 读写文件
RandomAccessFile既可以读文件，也可以写，所以它既包含了完全类似于InputStream的三个read()方法，其用法和InputStream的三个read()方法完全一样；也包含了完全类似于OutputStream的三个write()方法，其用法和OutputStream的三个write()方法完全一样。除此之外，RandomAccessFile还包含了一系列的readXxx()和writeXxx()方法来完成输入、输出。

RandomAccessFile不能直接向文件的指定位置插入内容，如果直接将文件记录指针移动到中间某位置后开始输出，则新输出的内容会覆盖文件中原有的内容。如果需要向指定位置插入内容，程序需要先把插入点后面的内容读入缓冲区，等把需要插入的数据写入文件后，再将缓冲区的内容追加到文件后面。

#### 构造方法
RandomAccessFile类有两个构造器，一个使用String参数来指定文件名，一个使用File参数来指定文件本身。除此之外，创建RandomAccessFile对象时，还需要指定一个mode参数，该参数指定RandomAccessFile的访问模式，该参数有如下4个值：
* "r"：以只读的方式打开指定文件。如果试图对该RandomAccessFile执行写入方法，都将抛出IOException异常。
* "rw"：以读、写方式打开指定文件。如果该文件尚不存在，则尝试创建该文件。
* "rwd"：以读、写方式打开指定文件。相对于"rw"模式，还要求对文件的内容的每个更新都同步写入到底层存储设备。
* "rws"：以读、写方式打开指定文件。相对于"rw"模式，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备。