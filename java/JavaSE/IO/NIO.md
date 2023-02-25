# NIO（New Input/Output）
新IO采用内存内存映射文件的方式来处理输入/输出，新IO将文件或文件的一段区域映射到内存中，像访问内存一样来访问文件（模拟了虚拟内存的概念）。新IO是面向块/缓存的。

Channel（通道）和Buffer（缓冲）是新IO中的两个核心对象：
* Channel是对传统的输入/输出系统的模拟，在新IO系统中所有的数据都需要通过通道传输。
* Buffer可以被理解成一个容器，它的本质是一个数组，发送到Channel中的所有对象都必须首先放到Buffer中，而从Channel中读取的数据也必须先放到Buffer中。

## Buffer

### 实现类
Buffer是一个抽象类，其最常用的子类是ByteBuffer，它可以在底层字节数组上进行get/set操作。除了ByteBuffer之外，对应于其他基本数据类型（boolean除外）都有相应的Buffer类：CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。

这些Buffer类都没有提供构造器，通过使用`static XxxBuffer allocate(int capacity)`方法创建一个容量为capacity的XxxBuffer对象。

其中ByteBuffer类还有一个子类：MappedByteBuffer，它用于表示Channel将磁盘文件的部分或全部内容映射到内存中后得到的结果，通常MappedByteBuffer对象由Channel的map()方法返回。

### 重要概念
在Buffer中有三个重要的概念：容量（capacity）、界限（limit）和位置（position）。
* 容量（capacity）：缓冲区的容量（capacity）表示该Buffer的最大数据容量，即最多可以存储多少数据。缓冲区的容量不可能为负值，创建后不能改变。
* 界限（limit）：第一个不应该被读出或者写入的缓冲区位置索引。也就是说，位于limit后的数据既不可被读，也不可被写。
* 位置（position）：用于指明下一个可以被读出的或者写入的缓冲区位置索引（类似于IO流中的记录指针）。当使用Buffer从Channel中读取数据时，position的值恰好等于已经读到了多少数据。当刚刚新建一个Buffer对象时，其position为0；如果从Channel中读取了2个数据到该Buffer中，则position为2，指向Buffer中第3个（第1个位置的索引为0）位置。

除此之外，Buffer里还支持一个可选的标记（mark，类似于传统IO流中的mark），Buffer允许直接将position定位到该mark处。这些值满足如下关系：0 ≤ mark ≤ position ≤ limit ≤ capacity。

![NIO+20230225123020](https://raw.githubusercontent.com/loli0con/picgo/master/images/NIO%2B20230225123020.png%2B2023-02-25-12-30-21)

### 方法
Buffer的主要作用就是装入数据，然后输出数据（其作用类似于前面介绍的取水的“竹筒”），开始时Buffer的position为0，limit为capacity，程序可通过put()方法向Buffer中放入一些数据（或者从Channel中获取一些数据），每放入一些数据，Buffer的position相应地向后移动一些位置。

当Buffer装入数据结束后，调用Buffer的flip()方法，该方法将limit设置为position所在位置，并将position设为0，这就使得Buffer的读写指针又移到了开始位置。也就是说，Buffer调用flip()方法之后，Buffer为输出数据做好准备；当Buffer输出数据结束后，Buffer调用clear()方法，clear()方法不是清空Buffer的数据，它仅仅将position置为0，将limit置为capacity，这样为再次向Buffer中装入数据做好准备。

除此之外，Buffer还包含如下一些常用的方法：
* int capacity()：返回Buffer的capacity大小。
* boolean hasRemaining()：判断当前位置（position）和界限（limit）之间是否还有元素可供处理。
* int limit()：返回Buffer的界限（limit）的位置。
* Buffer limit(int newLt)：重新设置界限（limit）的值，并返回一个具有新的limit的缓冲区对象。
* Buffer mark()：设置Buffer的mark位置，它只能在0和位置（position）之间做mark。
* int position()：返回Buffer中的position值。
* Buffer position(int newPs)：设置Buffer的position，并返回position被修改后的Buffer对象。
* int remaining()：返回当前位置和界限（limit）之间的元素个数。
* Buffer reset()：将位置（position）转到mark所在的位置。
* Buffer rewind()：将位置（position）设置成0，取消设置的mark。

除这些移动position、limit、mark的方法之外，Buffer的所有子类还提供了两个重要的方法：put()和get()方法，用于向Buffer中放入数据和从Buffer中取出数据。当使用put()和get()方法放入、取出数据时，Buffer既支持对单个数据的访问，也支持对批量数据的访问（以数组作为参数）。当使用put()和get()来访问Buffer中的数据时，分为相对和绝对两种：
* 相对（Relative）：从Buffer的当前position处开始读取或写入数据，然后将位置（position）的值按处理元素的个数增加。
* 绝对（Absolute）：直接根据索引向Buffer中读取或写入数据，使用绝对方式访问Buffer里的数据时，并不会影响位置（position）的值。

通过allocate()方法创建的Buffer对象都是普通Buffer，ByteBuffer还提供了一个allocateDirect()方法来创建直接Buffer。直接Buffer的创建成本比普通Buffer的创建成本高，但直接Buffer的读取效率更高。

## Channel
Channel类似于传统的流对象，但是有两个主要的区别：
* Channel可以直接将指定文件的部分活着全部直接映射成Buffer。
* 程序不能直接访问Channel中的数据，包括读写都不行，Channel只能与Buffer进行交互。
  * 如果要从Channel中取得数据，必须先用Buffer从Channel中取出一些数据，然后让程序从Buffer中取出这些数据。
  * 如果要将程序中的数据写入Channel，一样先让程序将数据放入Buffer中，程序再将Buffer里的数据写入Channel中。

### 实例
所有的Channel都不应该通过构造器来直接创建，而是通过传统的节点InputStream、OutputStream的getChannel()方法来返回对应的Channel，不同的节点流获得的Channel不一样。

在RandomAccessFile中也包含了一个getChannel()方法，RandomAccessFile返回的FileChannel()是只读的还是读写的，则取决于RandomAccessFile打开文件的模式。

### 方法
Channel中最常用的三类方法是map()、read()和write()。
* map()方法用于将Channel对应的部分或者全部数据映射成ByteBuffer。map()方法的方法签名为：`MappedByteBuffer map（FileChannel.MapMode mode，long position，long size）`，第一个参数执行映射时的模式，分别有只读、读写等模式；而第二个、第三个参数用于控制将Channel的哪些数据映射成ByteBuffer。
* read()或write()方法用于从Buffer中读取或向Buffer中写入数据。虽然XxxChannel既可以读取也可以写入，但XxxInputStream获取的XxxChannel只能读，而XxxOutputStream获取的XxxChannel只能写。



## 文件锁
Java提供了FileLock来支持文件锁定功能，在FileChannel中提供的lock()/tryLock()方法可以获得文件锁FileLock对象，从而锁定文件。lock()和tryLock()方法存在区别：
* 当lock()试图锁定某个文件时，如果无法得到文件锁，程序将一直阻塞；
* 而tryLock()是尝试锁定文件，它将直接返回而不是阻塞，如果获得了文件锁，该方法则返回该文件锁，否则将返回null。

如果FileChannel只想锁定文件的部分内容，而不是锁定全部内容，则可以使用如下的lock()或tryLock()方法。
* lock(long position, long size, boolean shared)：对文件从position开始，长度为size的内容加锁，该方法是阻塞式的。
* tryLock(long position, long size, boolean shared)：非阻塞式的加锁方法。参数的作用与上一个方法类似。


当参数shared为true时，表明该锁是一个共享锁，它将允许多个进程来读取该文件，但阻止其他进程获得对该文件的排他锁。当shared为false时，表明该锁是一个排他锁，它将锁住对该文件的读写。直接使用lock()或tryLock()方法获取的文件锁是排他锁。程序可以通过调用FileLock的isShared来判断它获得的锁是否为共享锁。

### 注意事项
* 在某些平台上，文件锁仅仅是建议性的，并不是强制性的。这意味着即使一个程序不能获得文件锁，它也可以对该文件进行读写。
* 在某些平台上，不能同步地锁定一个文件并把它映射到内存中。
* 文件锁是由Java虚拟机所持有的，如果两个Java程序使用同一个Java虚拟机运行，则它们不能对同一个文件进行加锁。
* 在某些平台上关闭FileChannel时，会释放Java虚拟机在该文件上的所有锁，因此应该避免对同一个被锁定的文件打开多个FileChannel。

## 后记
http://zhongmingmao.me/2019/07/22/java-performance-io-model/  
http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html