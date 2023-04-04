# 多线程

## 基本概念

### 操作系统
操作系统是配置在计算机硬件上第一层**软件**，它是一组*控制和管理计算机硬件与软件资源*，*合理地对各类作业进行调度*，以及*方便用户*的**程序的集合**。

操作系统概念中的“作业”，即任务。一般来说，一个任务对应一个进程（当然一个进程也可以执行多个任务，一个任务也可以由多个进程共同完成）。

### 进程
我们都知道程序，一个程序是静态的（例如HelloWorld.java文件），通常是存放在外存中的（例如硬盘）。而当程序被调入内存中运行后，就成了进程。顾名思义，**进程就是进行中的程序**，它是个动态的概念。**进程是系统进行资源分配与调度的基本单位**。

每一个进程都有自己的独立的一块内存空间。例如我们同时运行多个HelloWorld程序，每个进程里面都有一个“Hello world!”字符串，它们处于三个不同的内存区域之中（由操作系统分配），不会互相影响。即使某个进程奔溃了，出错了，其他的也不会受到影响。

### 线程
* 一个任务可以分为多个子任务，这里的“子任务”就是“线程”。
* 一个任务包含一到多个子任务，对应的**一个进程包含一到多个线程**。

**操作系统调度的最小任务单位是线程**。常用的Windows、Linux等操作系统都采用抢占式多任务，如何调度线程完全由操作系统决定，程序自己不能决定什么时候执行，以及执行多长时间。

### 进程 vs 线程
现代的操作系统都支持多任务，多任务既可以由**多进程模式**实现，也可以由单进程内的**多线程模式**实现，还可以由**多进程＋多线程模式**实现。

和多线程相比，多进程的缺点在于：
* 创建进程比创建线程**开销大**，尤其是在Windows系统上；
* 进程间通信比线程间**通信慢**，因为线程间通信就是读写同一个变量，速度很快。

而多进程的优点在于：
* 多进程**稳定**性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程，而在多线程的情况下，任何一个线程崩溃会直接导致整个进程崩溃。

### Java多线程
Java语言内置了多线程支持：一个Java程序实际上是一个JVM进程，JVM进程用一个主线程来执行main()方法，在main()方法内部，我们又可以启动多个线程。此外，JVM还有负责垃圾回收的其他工作线程等。

### 举个🌰
这个例子屏蔽了很多细节，所以不完全正确，或者说有错误的部分！！！

🌰：
1. 假设有
   * 一个用Java编写的视频播放器软件“*J播*”
   * 一个视频“*赛马.MP4*”
   * 操作系统里面已安装好了JRE
2. 启动视频播放器
   1. 通过鼠标，告诉操作系统，我要打开视频播放器
   2. 操作系统启动一个JVM进程，*J播*运行在这个JVM里面
3. 使用*J播*播放*赛马.MP4*
   1. *J播*请求JVM，JVM请求操作系统
   2. 操作系统把硬盘里的*赛马.MP4*加载到内存里（正在看的那部分加载即可）
   3. 操作系统把内存里的这部分数据分配给JVM，同时会一定程度上占有这个资源（讲道理，你播放的时候，这个视频文件不能被删除）
   4. JVM把视频数据给*J播*，*J播*至少会有两个线程，一个线程要处理图像数据并输出到显示器，一个线程处理音频数据并输出到扬声器

在这里例子里面：
* 操作系统控制了处理器、内存、硬盘、显示器、扬声器、鼠标等所有硬件资源；
* JVM进程或者说*J播*进程要通过向操作系统申请才能获得并使用这些资源；
* JVM进程一定程度上占有*赛马.MP4*这个资源，这个资源由操作系统分配的；
* JVM进程里至少包含了两个线程去处理*赛马.MP4*数据，分别处理图像和音频；
* 这两个线程由操作系统进行调度，最终交由处理器执行。

#### 进一步
Jvm是运行在操作系统之上的，程序员和Java代码打交道（.java文件），Java文件和Java编译器打交道（.class文件），class文件和Java虚拟机打交道，Java虚拟机和操作系统打交道。因此Jvm提供的功能其实是操作系统这个爸爸给的。

### 线程的生命周期
![multithreading+20210609170620](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609170620.png%2B2021-06-09-17-06-21)

#### Java线程的生命周期
![multithreading+java线程状态](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2Bjava%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.jpeg%2B2021-06-15-15-27-38)

* 提问：[java线程运行怎么有第六种状态?](https://www.zhihu.com/question/56494969) 
* 简答：Java里（Thread.State）定义的6种状态是虚拟机状态，不反映任何操作系统的线程状态。

##### 状态详细说明
1. 初始状态(NEW)
   * 新创建了一个线程对象，但还没有调用start()方法。
2. 就绪状态(RUNNABLE之READY)
   1. 就绪状态只是说你有资格运行，调度程序(Cpu)没有挑选到你，你就永远是就绪状态。
   2. 调用线程的start()方法，此线程进入就绪状态。
   3. 当前线程sleep()方法结束，其他线程join()结束，等待用户输入完毕，某个线程拿到对象锁，这些线程也将进入就绪状态
   4. 当前线程时间片用完了，调用当前线程的yield()方法，当前线程进入就绪状态。
3. 运行中状态(RUNNABLE之RUNNING)
   * 线程调度程序从可运行池中选择一个线程作为当前线程时线程所处的状态。这也是线程进入运行状态的唯一的一种方式。
4. 阻塞状态(BLOCKED)
   * 阻塞状态是线程阻塞在进入synchronized关键字修饰的方法或代码块(获取锁)时的状态。
5. 等待(WAITING)
   * 处于这种状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态。
6. 超时等待(TIMED_WAITING)
   * 处于这种状态的线程不会被分配CPU执行时间，不过无须无限期等待被其他线程显示地唤醒，在达到一定时间后它们会自动唤醒。
7. 终止状态(TERMINATED)
   1. 当线程的run()方法完成时，或者主线程的main()方法完成时，我们就认为它终止了。

##### 分析
通过对比，可以发现——操作系统的线程状态和Java世界里面的线程状态存在关联，但并非简单的一一对应关系：
* Jvm没有区分READY和RUNNING两种状态
* 阻塞状态，在Java里被细分为三类：BLOCKED、WAITING、TIMED_WAITING

一个Java线程要跑起来，（如果存在）要抢到锁，（必经步骤）还要抢占到CPU资源。

### 硬线程 软线程
CPU中的线程和操作系统（OS）中的线程即不同，两者都叫Thread是因为他们都是调度的基本单位，软件操作系统调度的基本单位是OS的Thread，硬件的调度基本单位是CPU中的Thread。操作系统负责把它产生的软Thread调度到CPU中的硬Thread中去。

除此之外还有[green thread](http://www.voidcn.com/article/p-bxncifft-bge.html)的概念。

Java使用的是native thread，即操作系统中的线程。



## 创建并启动

### 继承Thread类
通过继承Thread类来创建并启动多线程的步骤如下：
1. 定义Thread类的子类，并重写该类的run()方法，该run()方法的方法体就代表了线程需要完成的任务。因此把run()方法称为线程执行体。
2. 创建Thread子类的实例，即创建了线程对象。
3. 调用线程对象的start()方法来启动该线程。
   
### 实现Runnable接口
实现Runnable接口来创建并启动多线程的步骤如下：
1. 定义Runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。
2. 创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
3. 调用线程对象的start()方法来启动该线程。

Runnable对象仅仅作为Thread对象的target，Runnable实现类里包含的run()方法仅作为线程执行体。而实际的线程对象依然是Thread实例，只是该Thread线程负责执行其target的run()方法。

### 使用Callable和Future
Java5开始，提供了Callable接口，该接口是Runnable接口的增强版，提供了一个call()方法可以作为线程的执行体。（和run()方法对比）该方法可以**有返回值**，可以**声明抛出异常**。

Java5提供了Future接口来代表Callable接口里call()方法的返回值，并为Future接口提供了一个FutureTask实现类，该实现类实现了Future接口，并实现了Runnable接口——可以作为Thread类的target。

Callable接口有泛型限制，Callable接口里的泛型形参类型与call()方法返回值类型相同。而且Callable接口是函数式接口，因此可使用Lambda表达式创建Callable对象。

#### 方法
在Future接口里定义了如下几个公共方法来控制它关联的Callable任务：
* boolean cancel(boolean mayInterruptIfRunning)：试图取消该Future里关联的Callable任务。
* V get()：返回Callable任务里call()方法的返回值。调用该方法将导致程序阻塞，必须等到子线程结束后才会得到返回值。
* V get(long timeout, TimeUnit unit)：返回Callable任务里call()方法的返回值。该方法让程序最多阻塞timeout和unit指定的时间，如果经过指定时间后Callable任务依然没有返回值，将会抛出TimeoutException异常。
* boolean isCancelled()：如果在Callable任务正常完成前被取消，则返回true。
* boolean isDone()：如果Callable任务已完成，则返回true。

#### 步骤
创建并启动有返回值的线程的步骤如下：
1. 创建Callable接口的实现类，并实现call()方法，该call方法将作为线程执行体，且该方法有返回值，再创建Callable实现类的实例。
2. 使用FutureTask类来包装Callable对象，该Future对象封装了该Callable对象的call()方法的返回值。
3. 使用FutureTask对象作为Thread对象的target创建并启动新线程。
4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值。

### 对比
![multithreading+20210613171801](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210613171801.png%2B2021-06-13-17-18-02)

### 常用方法
* Thread.currentThread()：得到当前线程。
* thread.getName()：获取线程的名称。
* thread.setName()：为线程设置一个名称。
* thread.isAlive()：判断一个线程是否存活。


## 控制线程
### join线程
Thread提供了让一个线程等待另一个线程完成的方法——join()方法。当在某个程序执行流中调用其他线程的join()方法时，调用线程将被阻塞，直到被join()方法加入的join线程执行完成为止。join()方法有如下三种重载形式：
* join()：等待被join的线程执行完成。
* join(long millis)：等待被join的线程的时间最长为millis毫秒。如果在millis毫秒内被join的线程还没有执行结束，则不再等待。
* join(long millis, int nanos)：等待被join的线程的时间最长为millis毫秒加nanos毫微秒。

### 线程睡眠
如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过调用Thread类的静态sleep()方法来实现。线程在其睡眠时间内，不会获得执行机会，即使系统中没有其他可执行的线程。

### 线程让步
Thread类的yield()静态方法，也可以让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程转入就绪态ready。

当某个线程调用了yield()方法暂停之后，只有优先级与当前线程相同或更高的、处于就需状态的线程才会获得执行的机会。

该方法可能“没效果”——某线程让步了但又立刻获得执行的机会。

### 睡眠和让步的区别
sleep()方法和yield()方法的区别如下：
* sleep()方法暂停当前线程后，会给其他线程执行机会，不会理会其他线程的优先级；但yield()方法只会给优先级相同，或优先级更高的线程执行机会。
* sleep()方法会将线程转入阻塞状态，直到经过阻塞时间才会转入就绪状态；而yield()不会将线程转入阻塞状态，它只是强制当前线程进入就绪状态。因此完全有可能某个线程被yield()方法暂停之后，立即再次获得处理器资源被执行。
* sleep()方法声明抛出了InterruptedException异常，所以调用sleep()方法时要么捕捉该异常，要么显式声明抛出该异常；而yield()方法则没有声明抛出任何异常。
* sleep()方法比yield()方法有更好的可移植性，通常不建议使用yield()方法来控制并发线程的执行。

|区别|睡眠|让步|
|-----|-----|-----|
|优先级|**不理会**其他线程的优先级|只会给优先级**相同或更高**的就绪线程机会|
|状态切换|该线程进入**阻塞**|该线程进入**就绪**|
|throws|方法声明抛出了异常|没有声明抛出任何异常|
|移植性|很好|一般|

### 中断线程
中断有两种，正常/沟通的中断，和不正常/暴力的中断，就类似平时给电脑关机。  
这里主要描述正常中断，暴力中断的stop方法请永远不要使用它。

中断一个线程要在其他线程中对目标线程调用interrupt()方法，目标线程需要反复检测自身状态是否是interrupted状态（中断标识值是否为true），如果是，就立刻结束运行。要注意，interrupt()方法仅仅向目标线程发出了“中断请求”，至于目标线程是否能立刻响应，要看具体代码（这个过程需要程序员实现）。

中断只会影响wait状态、sleep状态和join状态，进入/处于上述三种状态，就会被影响。例如目标线程调用了join()方法，它正处于join状态，如果此时对目标线程调用interrupt()方法，那么它的join()方法会抛出InterruptedException异常。 
因此，当线程捕获到InterruptedException异常，就说明有其他线程对其调用了interrupt()方法，通常情况下该线程应该立刻结束运行。

### 后台线程
后台线程，又被称为守护线程，它是在后台运行的，它的任务是为其他的线程提供服务。

如果所有的前台线程都死亡，后台线程会自动死亡。当整个虚拟机中只剩下后台线程时，虚拟机就退出了。

调用Thread对象的setDaemon(true)方法可将指定线程设置成后台线程。前台线程创建的子线程默认是前台线程，后台线程创建的子线程默认是后台线程。Thread还提供了一个isDaemon()方法，用于判断指定线程是否为后台线程。

### 优先级
每个线程执行时都具有一定的优先级，优先级高的线程获得较多的执行机会，而优先级低的线程则获得较少的执行机会。每个线程默认的优先级都与创建它的父线程的优先级相同，在默认情况下，main线程具有普通优先级，由main线程创建的子线程也具有普通优先级。

Thread类提供了setPriority(int newPriority)、getPriority()方法来设置和返回指定线程的优先级，其中setPriority()方法的参数可以是一个整数，范围是1～10之间，也可以使用Thread类的如下三个静态常量。
* MAX_PRIORITY：其值是10。
* MIN_PRIORITY：其值是1。
* NORM_PRIORITY：其值是5。

考虑到移植性，通常只会使用上述的三个静态常量作为优先级。

## 线程同步

### 基本概念
#### 同步
先看看同步的汉语定义
![multithreading+20210615204214](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210615204214.png%2B2021-06-15-20-42-15)
再看看在计算机领域中的定义
![multithreading+20210615204325](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210615204325.png%2B2021-06-15-20-43-25)

线程在系统内部运行，经常需要协作完成任务。Java提供了一些机制来保证**线程协调运行**。

多线程协调运行的原则就是：当条件不满足时，线程进入等待状态；当条件满足时，线程被唤醒，继续执行任务。

#### 通信
由于多线程共享地址空间和数据空间，所以多个线程间的通信是一个线程的数据可以直接提供给其他线程使用，而不必通过操作系统。

线程间的通信目的主要是用于线程同步。


### 线程安全问题
多个线程操作（修改）同一个数据，会出现数据不一致的问题。

这是因为对变量进行读取和写入时，结果要正确，必须保证是原子操作。  
原子操作是指不能被中断的一个或一系列操作。

例如，对于语句：
```Java
n = n + 1;
```
看上去是一行语句，实际上对应了3条指令：
```
ILOAD   -->  取值
IADD    -->  运算
ISTORE  -->  写回
```
我们假设n的值是100，如果两个线程同时执行n = n + 1，得到的结果很可能不是102，而是101，原因在于：
```
┌───────┐    ┌───────┐
│Thread1│    │Thread2│
└───┬───┘    └───┬───┘
    │            │
    │ILOAD (100) │
    │            │ILOAD (100)
    │            │IADD
    │            │ISTORE (101)
    │IADD        │
    │ISTORE (101)│
    ▼            ▼
```
如果线程1在执行ILOAD后被操作系统中断，此刻如果线程2被调度执行，它执行ILOAD后获取的值仍然是100，最终结果被两个线程的ISTORE写入后变成了101，而不是期待的102。

这说明多线程模型下，要保证逻辑正确，对共享变量进行读写时，必须保证一组指令以原子方式执行：即某一个线程执行时，其他线程必须等待：
```
┌───────┐     ┌───────┐
│Thread1│     │Thread2│
└───┬───┘     └───┬───┘
    │             │
    │-- lock --   │
    │ILOAD (100)  │
    │IADD         │
    │ISTORE (101) │
    │-- unlock -- │
    │             │-- lock --
    │             │ILOAD (101)
    │             │IADD
    │             │ISTORE (102)
    │             │-- unlock --
    ▼             ▼
```

通过加锁和解锁的操作，就能保证3条指令总是在一个线程执行期间，不会有其他线程会进入此指令区间。即使在执行期线程被操作系统中断执行，其他线程也会因为无法获得锁导致无法进入此指令区间。只有执行线程将锁释放后，其他线程才有机会获得锁并执行。这种加锁和解锁之间的代码块我们称之为临界区（Critical Section），任何时候临界区最多只有一个线程能执行。可见，保证一段代码的原子性就是通过加锁和解锁实现的。

当然如果`n = n + 1`是原子操作，那么它就不需要同步了。

#### 处理方法
保证以下三个性质存在：
1. **原子性**：把一堆操作包裹起来当成一个操作
2. **可见性**：其他线程对某个值得修改是其他线程可见并且实时通知修改的
3. **有序性**：（临界区）保证串行，不允许并行

### 管程
![multithreading+20210615212552](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210615212552.png%2B2021-06-15-21-25-53)
上述是[管程的解释](https://segmentfault.com/a/1190000016417017)，其中多次提到“原语”，这里需要有一个基本的了解。
![multithreading+20210615213303](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210615213303.png%2B2021-06-15-21-33-04)
![multithreading+20210615213627](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210615213627.png%2B2021-06-15-21-36-28)
想了解更多关于jdk中的同步原理，可以阅读[这篇文章](https://www.cnblogs.com/binarylei/p/12544002.html)。

在这里，我们需要明白的是**管程是编程语言*在操作系统提供的同步原语的基础上*提出的更高层次的同步原语**。

### 同步监视器-概念
在Java中，**同步监视器即是对管程实现**，也简称为监视器。

### 同步代码块
为解决线程安全问题，Java的多线程引入了同步监视器。使用同步监视器的通用方法就是同步代码块，语法格式如下：
```Java
synchronized(obj){
    ...
    // 此处的代码就是同步代码块
}
```
上面的语法格式中synchronized后括号里的obj就是同步监视器，上面代码含义是：线程开始执行同步代码块之前，必须获得对同步监视器的锁定。通常推荐使用可能被并发访问的共享资源充当同步监视器。

任何时刻只有一个线程可以获得对同步监视器的锁定，当同步代码块执行完成后，该线程会释放对该同步监视器的锁定。

使用synchronized解决了多线程同步访问共享变量的正确性问题。但是，它的缺点是带来了性能下降。因为synchronized代码块无法并发执行。此外，加锁和解锁需要消耗一定的时间，所以，synchronized会降低程序的执行效率。

#### 标准用法
1. 找出修改共享变量的线程代码块；
2. 选择一个共享实例作为监视器；
3. 使用synchronized(lockObject) { ... }。

### 同步方法
同步方法就是使用synchronized关键字来修饰的方法，实例方法的同步监视器是this，静态方法的同步监视器是该类对应的Class对象。

### 同步监视器-详解
Java中的每个对象都有一个监视器，来监测并发代码的重入。在非多线程编码时该监视器不发挥作用，反之如果在synchronized范围内，监视器发挥作用。

![multithreading+20210616154831](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210616154831.png%2B2021-06-16-15-48-32)

一个线程（或者说一个执行流）在进入同步区（synchronized）时，需要获取*acquire*这个对象的监视器，即成为该监视器的拥有者*Owner*；如果一个线程在进入同步区时，发现监视器已被其他线程占有，它会在入口*entry*处等待其他线程释*release*放监视器。

已经获得监视器的线程可以调用wait()方法，释放它所拥有的监视器，进入等待*wait*区；其他进入同步区的线程可以向等待区的线程发通知notify()，以此来唤醒它们。

这里再放一个线程转换状态图，重点关注图中的“等待队列”和“锁池状态”。
![multithreading+20210616161136](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210616161136.png%2B2021-06-16-16-11-37)


### 释放同步监视器的锁定
程序无法显式释放对同步监视器的锁定，线程会在如下几种情况下释放对同步监视器的锁定：
* **离开同步块**
  * 当前线程的同步方法、同步代码块执行结束，当前线程即释放同步监视器。
  * 当前线程在同步代码块、同步方法中遇到break、return终止了该代码块、该方法的继续执行，当前线程将会释放同步监视器。
  * 当前线程在同步代码块、同步方法中出现了未处理的Error或Exception，导致了该代码块、该方法异常结束时，当前线程将会释放同步监视器。
* 当前线程执行同步代码块或同步方法时，程序执行了同步监视器对象的**wait()方法**，则当前线程暂停，并释放同步监视器。


### 同步锁
同步锁Lock是控制多个线程对共享资源进行访问的工具，它比同步代码块和同步方法更灵活，每次只能由一个线程对Lock对象加锁，线程开始访问共享资源之前应该先获得Lock对象。访问结束之后，必须释放锁（通过finally）。

Lock、ReadWriteLock是两个根接口，ReentrantLock（可重入锁）、ReentrantReadWriteLock分辨是它们的实现类。  
还有更强大的一个StampedLock类，它为读写操作提供了三种锁模式：Writing、ReadingOptimistic、Reading。

#### 锁和监视器
[锁为实现监视器提供必要的支持](https://www.jianshu.com/p/ed9e616bfad3)。  
详细可阅读上面文章，目前可以简单地把“锁”和“监视器”两个概念画上等号。

#### 常用锁介绍

##### ReentrantLock
JVM允许同一个线程重复获取同一个锁，这种能被同一个线程反复获取的锁，就叫做可重入锁——ReentrantLock。  
ReentrantLock是Java里最基本的锁。

##### ReadWriteLock
读写锁是计算机程序的并发控制的一种同步机制：读操作可并发重入，写操作是互斥的。

使用ReadWriteLock可以提高读取效率：
* ReadWriteLock只允许一个线程写入；
* ReadWriteLock允许多个线程在没有写入时同时读取；
* ReadWriteLock适合读多写少的场景

[ReadWriteLock用法](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581002092578)

##### StampedLock
StampedLock提供了乐观读锁，可取代ReadWriteLock以进一步提升并发性能；StampedLock是不可重入锁。

[StampedLock用法](https://www.liaoxuefeng.com/wiki/1252599548343744/1309138673991714)

##### TODO 更多🔒

### 死锁
死锁产生的条件是多线程各自持有不同的锁，并互相试图获取对方已持有的锁，导致无限等待；避免死锁的方法是多线程获取锁的顺序要一致。


## 线程通信

### Object的wait和notify
Object类提供了wait()、notify()和notifyAll三个方法，可让同步监视器对象来进行调用，以此实现线程通信。
* wait()：导致当前线程等待，直到其他线程调用该同步监视器的notify()方法或notifyAll()方法来唤醒该线程。该wait()方法有三种形式——无时间参数的wait（一直等待，直到其他线程通知）、带毫秒参数的wait()和带毫秒、毫微秒参数的wait()（这两种方法都是等待指定时间后自动苏醒）。调用wait()方法的当前线程会释放对该同步监视器的锁定。
* notify()：唤醒在此同步监视器上等待的单个线程。如果所有线程都在此同步监视器上等待，则会选择唤醒其中一个线程。选择是任意性的。只有当前线程放弃对该同步监视器的锁定后（使用wait()方法），才可以执行被唤醒的线程。
* notifyAll()：唤醒在此同步监视器上等待的所有线程。只有当前线程放弃对该同步监视器的锁定后，才可以执行被唤醒的线程。

这种通信方式主要用于同步代码块/同步方法。

### Condition
当使用Lock对象来保证同步时，Java提供了一个Condition类来保持协调，使用Condition可以让那些已经得到Lock对象却无法继续执行的线程释放Lock对象，Condition对象也可以唤醒其他处于等待的线程。

Condition实例被绑定在一个Lock对象上，通过调用Lock对象的newCondition()方法即可获得。  
Condition类提供了如下三个方法：
* await()：类似于隐式同步监视器上的wait()方法，导致当前线程等待，直到其他线程调用该Condition的signal()方法或signalAll()方法来唤醒该线程。该await()方法有更多变体，如long awaitNanos（long nanosTimeout）、void awaitUninterruptibly()、awaitUntil（Date deadline）等，可以完成更丰富的等待操作。
* signal()：唤醒在此Lock对象上等待的单个线程。如果所有线程都在该Lock对象上等待，则会选择唤醒其中一个线程。选择是任意性的。只有当前线程放弃对该Lock对象的锁定后（使用await()方法），才可以执行被唤醒的线程。
* signalAll()：唤醒在此Lock对象上等待的所有线程。只有当前线程放弃对该Lock对象的锁定后，才可以执行被唤醒的线程。
  
### 阻塞队列
Java 5提供了一个BlockingQueue接口，BlockingQueue具有一个特征：当生产者线程试图向BlockingQueue中放入元素时，如果该队列已满，则该线程被阻塞；当消费者线程试图从BlockingQueue中取出元素时，如果该队列已空，则该线程被阻塞。程序的两个线程通过交替向BlockingQueue中放入元素、取出元素，即可很好地控制线程的通信。

BlockingQueue包含的方法：
||抛出异常|（根据成败与否）不同返回值|阻塞线程|指定超时时长|
|---|---|---|---|---|
|队尾插入元素|add(e)|offer(e)|put(e)|put(e,time,unit)|
|队头删除元素|remove()|poll()|take()|poll(time,unit)|
|获取但不删除元素|element()|peek()|无|无|

![multithreading+20230403222131](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20230403222131.png%2B2023-04-03-22-21-34)
BlockingQueue包含如下5个实现类：
* ArrayBlockingQueue：基于数组实现的BlockingQueue队列。
* LinkedBlockingQueue：基于链表实现的BlockingQueue队列。
* PriorityBlockingQueue：优先队列，该队列调用remove()、poll()、take()等方法取出元素时，取出的是队列中最小的元素。PriorityBlockingQueue判断元素的大小即可根据元素（实现Comparable接口）的本身大小来自然排序，也可使用Comparator进行定制排序。
* SynchronousQueue：同步队列。对该队列的存、取操作必须交替进行。
* DelayQueue：它是一个特殊的BlockingQueue，底层基于PriorityBlockingQueue实现。不过，DelayQueue要求集合元素都实现Delay接口（该接口里只有一个long getDelay()方法），DelayQueue根据集合元素的getDalay()方法的返回值进行排序。

### 共享变量 与 volatile
线程间共享变量需要使用volatile关键字标记，确保每个线程都能读取到更新后的变量值。

在Java虚拟机中，变量的值保存在主内存中，但是，当线程访问变量时，它会先获取一个副本，并保存在自己的工作内存中。如果线程修改了变量的值，虚拟机会在某个时刻把修改后的值回写到主内存，但是，这个时间是不确定的！这会导致如果一个线程更新了某个变量，另一个线程读取的值可能还是更新前的。

因此，volatile关键字的目的是告诉虚拟机：
* 每次访问变量时，总是获取主内存的最新值；
* 每次修改变量后，立刻回写到主内存。

volatile关键字解决的是可见性问题：当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值。




## 线程组 和 未处理的异常
Java使用ThreadGroup来表示线程组，它可以对一批线程进行分类管理。

用户创建的所有线程都属于指定线程组，如果没有显式指定线程属于哪个线程组，则该线程属于默认线程组。在默认情况下，子线程和创建它的父线程初一同一个线程组内。一旦某个线程加入了指定线程组之后，该线程将一直属于该线程组，直到该线程死亡，线程运行中途不能改变它所属的线程组。因为中途不可改变线程所属的线程组，所以Thread类没有提供setThreadGroup()方法来改变线程所属的线程组，但提供了一个getThreadGroup()方法来返回该线程所属的线程组，getThreadGroup()方法的返回值是ThreadGroup对象，表示一个线程组。

Thread类提供了如下几个构造器来设置新创建的线程属于哪个线程组：
* Thread(ThreadGroup group, Runnable target)：以target的run()方法作为线程执行体创建新线程，属于group线程组。
* Thread(ThreadGroup group, Runnable target, String name)：以target的run()方法作为线程执行体创建新线程，该线程属于group线程组，且线程名为name。
* Thread(ThreadGroup group, String name)：创建新线程，新线程名为name，属于group线程组。

ThreadGroup类提供了如下两个简单的构造器来创建实例：
* ThreadGroup(String name)：以指定的线程组名字来创建新的线程组。
* ThreadGroup(ThreadGroup parent, String name)：以指定的名字、指定的父线程组创建一个新线程组。

ThreadGroup类提供了如下几个常用的方法来操作整个线程组里的所有线程：
* int activeCount()：返回此线程组中活动线程的数目。
* interrupt()：中断此线程组中的所有线程。
* isDaemon()：判断该线程组是否是后台线程组。
* setDaemon(boolean daemon)：把该线程组设置成后台线程组。后台线程组具有一个特征——当后台线程组的最后一个线程执行结束或最后一个线程被销毁后，后台线程组将自动销毁。
* setMaxPriority(int pri)：设置线程组的最高优先级。

### 异常处理
在Java中，线程中的异常是不能抛出到调用该线程的外部方法中捕获的。

因为线程是独立执行的代码片断，线程的问题应该由线程自己来解决，而不要委托到外部。基于这样的设计理念，在Java中，线程方法的异常都应该在线程代码边界之内（run方法内）进行try-catch并处理掉。

```Java
// Runnable接口
public interface Runnable {
    public abstract void run();  // 没有throws，不允许抛出异常
}

// Thread内部的UncaughtExceptionHandler接口
public interface UncaughtExceptionHandler {
        void uncaughtException(Thread t, Throwable e);
    }
```
通过Runnable接口的定义，我们可以看出，run()方法里的Checked Exception需要我们去捕获。

在线程中抛出了异常，线程会即刻终结，而且在主线程中并不能捕获到这个异常。  
除了使用try-catch来包含代码块，Java还提供另种一种方式处理异常——使用UncaughtExceptionHandler，中文翻为“未捕获异常处理器”。UncaughtExceptionHandler是Thread的内部接口。

![multithreading+20210618172617](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210618172617.png%2B2021-06-18-17-26-18)
Thread内部有两个UncaughtExceptionHandler变量，用于记录异常处理器：
* defaultUncaughtExceptionHandler是属于类的，也就是整个程序使用的，也叫默认处理器。
* uncaughtExceptionHandler是属于实例的，也就是某个线程使用的。

针对上述的变量分别有getter和setter方法；setter方法都是直接设置，getDefaultUncaughtExceptionHandler是直接获取，getUncaughtExceptionHandler如果非空那么直接获取，否则将会返回当前线程组。

线程组实现了Thread.UncaughtExceptionHandler，内部实现了方法public void uncaughtException(Thread t, Throwable e)。
换句话说，线程组内部实现了一个处理器，能处理组内线程的未捕获异常。

线程组内部的处理器的处理逻辑如下：
1. 查看父线程组是否重写了uncaughtException，如是则调用父线程组的方法（注意这个过程会一直递归到顶级线程组），如否则*2.*
2. 查看默认处理器是否存在，如是则调用默认处理器的方法，如否则*3.*
3. （如果该异常对象不是ThreadDeath的实例，）在system.err打印错误信息

![multithreading+20210618174154](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210618174154.png%2B2021-06-18-17-41-55)

## 线程池
系统启动一个新线程的成本是比较高的，当程序中需要创建大量生存期很短暂的线程时，应该考虑实现线程池。除此之外，使用线程池可以有效控制系统中并发线程的数量。

Java标准库提供了ExecutorService接口表示线程池.

线程池在系统启动时即创建大量空闲的线程，程序讲一个Runnable对象或者Callable对象传给线程池，线程池就会启动一个空闲的线程来执行它们的run()或call()方法，当run()或call()方法执行结束后，该线程并不会死亡，而是再次返回线程池中成为空闲状态，等待执行下一个Runnable对象的run()或call()方法。

ExecutorService只是接口，Java标准库提供的几个常用实现类有：
* FixedThreadPool：线程数固定的线程池；
* CachedThreadPool：线程数根据任务动态调整的线程池；
* SingleThreadExecutor：仅单线程执行的线程池；
* ScheduledThreadPool：可以定期调度多个任务的线程池。
### 创建线程池
Java使用Executors工厂类来产生线程池，该工程类包含如下几个静态工厂方法来创建线程池：
* newCachedThreadPool()：创建一个具有缓存功能的线程池，系统根据需要创建线程，这些线程将会被缓存在线程池中。
* newFixedThreadPool(int nThreads)：创建一个可重用的、具有固定线程数的线程池。
* newSingleThreadExecutor()：创建一个只有单线程的线程池，它相当于调用newFixedThread Pool()方法时传入参数为1。
* newScheduledThreadPool(int corePoolSize)：创建具有指定线程数的线程池，它可以在指定延迟后执行线程任务。corePoolSize指池中所保存的线程数，即使线程是空闲的也被保存在线程池内。
* newSingleThreadScheduledExecutor()：创建只有一个线程的线程池，它可以在指定延迟后执行线程任务。
* ExecutorService newWorkStealingPool(int parallelism)：创建持有足够的线程的线程池来支持给定的并行级别，该方法还会使用多个队列来减少竞争。
* ExecutorService newWorkStealingPool()：该方法是前一个方法的简化版本。如果当前机器有4个CPU，则目标并行级别被设置为4，也就是相当于为前一个方法传入4作为参数。


### 提交任务
1. 前三个方法返回一个ExecutorService对象，该对象代表一个代表尽快执行线程的线程池（只要线程池中有空闲线程，就立即执行线程任务），程序只要将一个Runnable对象或Callable对象（代表线程任务）提交给该线程池，该线程池就会尽快执行该任务。ExecutorService里提供了如下三个方法：
   * Future<?> submit(Runnable task)：将一个Runnable对象提交给指定的线程池，线程池将在有空闲线程时执行Runnable对象代表的任务。其中Future对象代表Runnable任务的返回值——但run()方法没有返回值，所以Future对象将在run()方法执行结束后返回null。但可以调用Future的isDone()、isCancelled()方法来获得Runnable对象的执行状态。
   * <T> Future<T> submit(Runnable task, T result)：将一个Runnable对象提交给指定的线程池，线程池将在有空闲线程时执行Runnable对象代表的任务。其中result显式指定线程执行结束后的返回值，所以Future对象将在run()方法执行结束后返回result。
   * <T> Future<T> submit(Callable<T> task)：将一个Callable对象提交给指定的线程池，线程池将在有空闲线程时执行Callable对象代表的任务。其中Future代表Callable对象里call()方法的返回值。
2. 中间两个方法返回一个ScheduledExecutorService线程池，它是ExecutorService的子类，代表可在指定延迟后或周期性地执行线程任务的线程池，它提供了如下4个方法：
   * ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)：指定callable任务将在delay延迟后执行。
   * ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)：指定command任务将在delay延迟后执行。
   * ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)：指定command任务将在delay延迟后执行，而且以设定频率重复执行。也就是说，在initialDelay后开始执行，依次在initialDelay+period、initialDelay+2*period…处重复执行，依此类推。
   * ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)：创建并执行一个在给定初始延迟后首次启用的定期操作，随后在每一次执行终止和下一次执行开始之间都存在给定的延迟。如果任务在任一次执行时遇到异常，就会取消后续执行；否则，只能通过程序来显式取消或终止该任务。
3. 最后两个方法生成的work stealing池相当于后台线程池，如果所有的前台线程都死亡了，work stealing池中的线程会自动死亡。


### 关闭线程池
用完一个线程池后，应该调用该线程池的shutdown()方法，该方法将启动线程池的关闭序列，调用shutdown()方法后的线程池不再接收新任务，但会将以前所有已提交任务执行完成。当线程池中的所有任务都执行完成后，池中的所有线程都会死亡；另外也可以调用线程池的shutdownNow()方法来关闭线程池，该方法试图停止所有正在执行的活动任务，暂停处理正在等待的任务，并返回等待执行的任务列表。

线程池的awaitTermination()则会等待指定的时间让线程池关闭。

### 完整步骤
使用线程池来执行线程任务的步骤如下：
1. 调用Executors类的静态工厂方法创建一个ExecutorService对象，该对象代表一个线程池。
2. 创建Runnable实现类或Callable实现类的实例，作为线程执行任务。
3. 调用ExecutorService对象的submit()方法来提交Runnable实例或Callable实例。
4. 当不想提交任何任务时，调用ExecutorService对象的shutdown()方法来关闭线程池。

### ForkJoinPool
Java提供了ForkJoinPool来支持将一个任务拆分成多个“小任务”并行计算，再把多个“小任务”的结果合并成总的计算结果。ForkJoinPool是ExecutorService的实现类，因此是一种特殊的线程池。ForkJoinPool提供了如下两个常用的构造器：
* ForkJoinPool(int parallelism)：创建一个包含parallelism个并行线程的ForkJoinPool。
* ForkJoinPool()：以Runtime.availableProcessors()方法的返回值作为parallelism参数来创建Fork JoinPool。

Java为ForkJoinPool增加了通用池功能，ForkJoinPool类通过如下两个静态方法提供通用池功能：
* ForkJoinPool commonPool()：该方法返回一个通用池，通用池的运行状态不会受shutdown()或shutdownNow()方法的影响。当然，如果程序直接执行System.exit(0)；来终止虚拟机，通用池以及通用池中正在执行的任务都会被自动终止。
* int getCommonPoolParallelism()：该方法返回通用池的并行级别。


创建了ForkJoinPool实例之后，就可调用ForkJoinPool的`submit(ForkJoinTask task)`或`invoke(ForkJoinTask task)`方法来执行指定任务了。其中ForkJoinTask代表一个可以并行、合并的任务。ForkJoinTask是一个抽象类，它还有两个抽象子类：RecursiveAction和RecursiveTask。其中RecursiveTask代表有返回值的任务，而RecursiveAction代表没有返回值的任务。

ForkJoinPool、ForkJoinTask等类的类图：
![multithreading+20230404114012](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20230404114012.png%2B2023-04-04-11-40-14)


## 线程相关类

### ThreadLocal类
Java为多线程编程提供了一个ThreadLocal类，通过使用ThreadLocal类可以简化多线程编程时的并发访问，使用这个工具类可以很简捷地隔离多线程程序的竞争资源。

ThreadLocal，是Thread Local Variable（线程局部变量）的意思。线程局部变量（ThreadLocal）的功用其实非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，使每一个线程都可以独立地改变自己的副本，而不会和其他线程的副本冲突，从根本上避免多个线程之间对共享资源（变量）的竞争，也就不需要对多个线程进行同步了。ThreadLocal类只提供了如下三个public方法：
* T get()：返回此线程局部变量中当前线程副本中的值。
* void remove()：删除此线程局部变量中当前线程的值。
* void set(T value)：设置此线程局部变量中当前线程副本中的值。

### 原子类
Java的java.util.concurrent包提供了一组原子操作的封装类，它们位于java.util.concurrent.atomic包。  
Atomic类是通过无锁（lock-free）的方式实现的线程安全（thread-safe）访问。它的主要原理是利用了CAS：Compare and Set。

[原子类详解1](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581083881506)  
[原子类详解2](https://www.cnblogs.com/jingmoxukong/p/12109049.html)

### CompletableFuture
CompletableFuture，它针对Future做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。  
CompletableFuture实现了Future和CompletionStage接口，前者使其具备了Future的所有功能，这里我们重点讨论后者——CompletionStage。

CompletionStage 代表异步计算中的一个阶段或步骤。该接口定义了多种不同的方式，将CompletionStage 实例与其他实例或代码链接在一起，比如完成时调用的方法（代码链接）。  
CompletionStage 的接口一般都返回新的CompletionStage，表示执行完一些逻辑后，生成新的CompletionStage，构成链式的阶段型的操作。

可以举一个简单的“[煮饭&干饭](https://juejin.cn/post/6844904195162636295)”的例子，来阐述CompletableFuture和CompletionStage。

[使用ComletableFuture可以将异步计算中处理流程进行串行化或并行化](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650)。


### Concurrent集合
![multithreading+20230404115049](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20230404115049.png%2B2023-04-04-11-50-52)

Java标准库的java.util.concurrent包提供了大量支持高效并发访问的集合接口和实现类：
|interface|non-thread-safe|thread-safe|
|-----|-----|-----|
|List|ArrayList|CopyOnWriteArrayList|
|Map|HashMap|ConcurrentHashMap|
|Set|HashSet / TreeSet|CopyOnWriteArraySet|
|Queue|ArrayDeque / LinkedList|ArrayBlockingQueue / LinkedBlockingQueue / ConcurrentLinkedQueue|
|Deque|ArrayDeque / LinkedList|LinkedBlockingDeque / ConcurrentLinkedDeque|

java.util.Collections工具类还提供了一个旧的线程安全集合转换器，但是它实际上是用一个包装类包装了非线程安全的集合，然后对所有读写方法都用synchronized加锁，这样获得的线程安全集合的性能比java.util.concurrent集合要低很多，所以不推荐使用。

线程安全的集合类可分为如下两类：
* 以Concurrent开头的集合类：以Concurrent开头的集合类代表了支持并发访问的集合，它们可以支持多个线程并发写入访问，这些写入线程的所有操作都是线程安全的，但读取操作不必锁定。以Concurrent开头的集合类采用了分段锁的算法来保证永远不会锁住整个集合，因此在并发写入时有较好的性能。
* 以CopyOnWrite开头的集合类：采用复制底层数组的方式来实现写操作。当线程对CopyOnWriteArrayList集合执行读取操作时，线程将会直接读取集合本身，无须加锁与阻塞。当线程对CopyOnWriteArrayList集合执行写入操作时（包括调用add()、remove()、set()等方法），该集合会在底层复制一份新的数组，接下来对新的数组执行写入操作。由于CopyOnWriteArrayList执行写入操作时需要频繁地复制数组，性能比较差，但由于读操作与写操作不是操作同一个数组，而且读操作也不需要加锁，因此读操作就很快、很安全。

### 发布-订阅框架
Java 提供了一个发布-订阅框架，该框架是基于异步响应流的。这个发布-订阅框架可以非常方便地处理异步线程之间的流数据交换（比如两个线程之间需要交换数据）。而且这个发布-订阅框架不需要使用数据中心来缓冲数据，同时具有非常高效的性能。这个发布-订阅框架使用Flow类的4个静态内部接口作为核心API：
* Flow.Publisher：代表数据发布者、生产者。
* Flow.Subscriber：代表数据订阅者、消费者。
* Flow.Subscription：代表发布者和订阅者之间的链接纽带。订阅者既可通过调用该对象的request()方法来获取数据项，也可通过调用对象的cancel()方法来取消订阅。
* Flow.Processor：数据处理器，它可同时作为发布者和订阅者使用。


Flow.Publisher发布者作为生产者，负责发布数据项，并注册订阅者。Flow.Publisher接口定义了如下方法来注册订阅者：
* void subscribe(Flow.Subscriber<?super T>subscriber)：程序调用此方法注册订阅者时，会触发订阅者的onSubscribe()方法，而Flow.Subscription对象作为参数传给该方法；如果注册失败，将会触发订阅者的onError() 方法。


Flow.Subscriber接口定义了如下方法：
* void onSubscribe(Flow.Subscription subscription)：订阅者注册时自动触发该方法。
* void onComplete()：当订阅结束时触发该方法。
* void onError(Throwable throwable)：当订阅失败时触发该方法。
* void onNext(T item)：订阅者从发布者处获取数据项时触发该方法，订阅者可通过该方法获取数据项。


为了处理一些通用发布者的场景，Java 为Flow.Publisher提供了一个SubmissionPublisher实现类，它可向当前订阅者异步提交非空的数据项，直到它被关闭。每个订阅者都能以相同的顺序接收到新提交的数据项。程序创建SubmissionPublisher对象时，需要传入一个线程池作为底层支撑；该类也提供了一个无参数的构造器，该构造器使用ForkJoinPool.commonPool()方法来提交发布者，以此实现发布者向订阅者提供数据项的异步特性。

## 后记
license的坑还没填，侵删🥺

优秀博文：
* https://www.cnblogs.com/wxd0108/p/5479442.html
* https://blog.csdn.net/Evankaka/article/details/44153709
* https://segmentfault.com/a/1190000023960592
* https://www.liaoxuefeng.com/wiki/1252599548343744/1255943750561472
* https://blog.csdn.net/qq_22771739/article/details/82529874
* https://www.cnblogs.com/noteless/p/10354733.html
* https://www.cnblogs.com/noteless/p/10394054.html
* https://blog.csdn.net/weixin_45970688/article/details/106693287
* https://zhuanlan.zhihu.com/p/70361897
* https://www.jianshu.com/p/cf57726e77f2
* https://blog.csdn.net/pange1991/article/details/82115437