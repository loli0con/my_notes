# 多线程

## 基本概念

### 操作系统
操作系统是配置在计算机硬件上第一层**软件**，它是一组*控制和管理计算机硬件与软件资源*，*合理地对各类作业进行调度*，以及*方便用户*的**程序的集合**。

操作系统概念中的“作业”，即任务。一般来说，一个任务对应一个进程（当然一个进程也可以执行多个任务，一个任务也可以由多个进程共同完成）。

### 进程
我们都知道程序，一个程序是静态的（例如HelloWorld.java文件），通常是存放在外存中的（例如硬盘）。而当程序被调入内存中运行后，就成了进程。  
顾名思义，**进程就是进行中的程序**，它是个动态的概念。**进程是系统进行资源分配与调度的基本单位**。

每一个进程都有自己的独立的一块内存空间。例如我们同时运行多个HelloWorld程序，每个进程里面都有一个“Hello world!”字符串，它们处于三个不同的内存区域之中（由操作系统分配），不会互相影响。即使某个进程奔溃了，出错了，其他的也不会受到影响。

### 线程
一个任务可以分为多个子任务，这里的“子任务”就是“线程”。  
一个任务包含一到多个子任务，**一个进程包含一到多个线程**。

**操作系统调度的最小任务单位是线程**。常用的Windows、Linux等操作系统都采用抢占式多任务，如何调度线程完全由操作系统决定，程序自己不能决定什么时候执行，以及执行多长时间。

### 进程 vs 线程
现代的操作系统都支持多任务，多任务既可以由多进程实现，也可以由单进程内的多线程实现，还可以混合多进程＋多线程。

和多线程相比，多进程的缺点在于：
* 创建进程比创建线程**开销大**，尤其是在Windows系统上；
* 进程间通信比线程间**通信慢**，因为线程间通信就是读写同一个变量，速度很快。

而多进程的优点在于：
* 多进程稳定性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程，而在多线程的情况下，任何一个线程崩溃会直接导致整个进程崩溃。

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
   2. 操作系统把硬盘里的*赛马.MP4*加载到内存里（不需要加载全部，部分加载即可）
   3. 操作系统把内存里的这部分数据分配给JVM，同时会一定程度上锁住这个资源（讲道理，你播放的时候，这个视频文件不能被删除)
   4. JVM把视频数据给*J播*，*J播*至少会有两个线程，一个线程要处理图像数据并输出到显示器，一个线程处理音频数据并输出到扬声器

在这里例子里面：
* 操作系统控制了处理器、内存、硬盘、显示器、扬声器、鼠标等所有硬件资源；
* JVM进程或者说*J播*进程要通过向操作系统申请才能获得并使用这些资源；
* 进程一定程度上占有*赛马.MP4*这个资源，这个资源由操作系统分配的；
* 进程里至少包含了两个线程去处理*赛马.MP4*数据，分别处理图像和音频；
* 这两个线程由操作系统进行调度，最终交由处理器执行。

### 线程的生命周期
![multithreading+20210609170620](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609170620.png%2B2021-06-09-17-06-21)

[java线程运行怎么有第六种状态?](https://www.zhihu.com/question/56494969)

### 硬线程 软线程
CPU中的线程和操作系统（OS）中的线程即不同，两者都叫Thread是因为他们都是调度的基本单位，软件操作系统调度的基本单位是OS的Thread，硬件的调度基本单位是CPU中的Thread。操作系统负责把它产生的软Thread调度到CPU中的硬Thread中去。

除此之外还有green thread。Java使用的是native thread，即操作系统中的线程。

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

![multithreading+20210609155657](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609155657.png%2B2021-06-09-15-56-58)

### 使用Callable和Future
Java5开始，提供了Callable接口，该接口是Runnable接口的增强版，提供了一个call()方法可以作为线程的执行体。该方法可以有返回值，可以声明抛出异常。

Java5提供了Future接口来代表Callable接口里call()方法的返回值，并为Future接口提供了一个FutureTask实现类，该实现类实现了Future接口，并实现了Runnable接口——可以作为Thread类的target。  
注意，Callable接口有泛型限制，Callable接口里的泛型形参类型与call()方法返回值类型相同。

![multithreading+20210609161204](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609161204.png%2B2021-06-09-16-12-04)
![multithreading+20210609161217](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609161217.png%2B2021-06-09-16-12-17)

创建并启动有返回值的线程的步骤如下：
1. 创建Callable接口的实现类，并实现call()方法，该call方法将作为线程执行体，且该方法有返回值，再创建Callable实现类的实例。
2. 使用FutureTask类来包装Callable对象，该Future对象封装了该Callable对象的call()方法的返回值。
3. 使用FutureTask对象作为Thread对象的target创建并启动新线程。
4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值。

### 对比
![multithreading+20210613171801](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210613171801.png%2B2021-06-13-17-18-02)

### 常用方法
Thread.currentThread()  
thread.getName()  
thread.setName()  

## 控制线程
### join线程
Thread提供了让一个线程等待另一个线程完成的方法——join()方法。当在某个程序执行流中调用其他线程的join()方法时，调用线程将被阻塞，直到被join()方法加入的join线程执行完成为止。
![multithreading+20210609171013](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609171013.png%2B2021-06-09-17-10-14)

### 后台线程
后台线程，又被称为守护线程，它是在后台运行的，它的任务是为其他的线程提供服务。

如果所有的前台线程都死亡，后台线程会自动死亡。  
当整个虚拟机中只剩下后台线程时，虚拟机就退出了。

调用Thread对象的setDaemon(true)方法可将指定线程设置成后台线程。前台线程创建的子线程默认是前台线程，后台线程创建的子线程默认是后台线程。  
Thread还提供了一个isDaemon()方法，用于判断指定线程是否为后台线程。

### 线程睡眠
如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过调用Thread类的静态sleep()方法来实现。线程在其睡眠时间内，不会获得执行机会，即使系统中没有其他可执行的线程。

### 线程让步
Thread类的yield()静态方法，也可以让当正在执行的前线程暂停，但它不会阻塞该线程，它只是将该线程转入就绪态。

### 睡眠和让步的区别
![multithreading+20210609174439](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609174439.png%2B2021-06-09-17-44-40)

### 优先级
![multithreading+20210609174621](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210609174621.png%2B2021-06-09-17-46-22)

## 线程同步

### 线程安全问题
多线程同时修改一个共享变量，会出现数据不一致的问题。

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

### 同步方法
同步方法就是使用synchronized关键字来修饰的方法，实例方法的同步监视器是this，静态方法的同步监视器是该类对应的Class对象。

### 释放同步监视器的锁定
![multithreading+WechatIMG70677](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2BWechatIMG70677.jpeg%2B2021-06-11-20-23-10)

### 同步锁
同步锁Lock是控制多个线程对共享资源进行访问的工具，它比同步代码块和同步方法更灵活，每次只能由一个线程对Lock对象加锁，线程开始访问共享资源之前应该先获得Lock对象。

### 死锁
当两个线程相互等待对方释放同步监视器时就会发生死锁。

## 线程通信

### Object
Object类提供了wait()、notify()和notifyAll三个方法，可让同步监视器对象来进行调用，以此实现线程通信。
![multithreading+20210611204234](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611204234.png%2B2021-06-11-20-42-35)

### Condition
当使用Lock对象来保证同步时，Java提供了一个Condition类来保持协调，使用Condition可以让那些已经得到Lock对象却无法继续执行的线程释放Lock对象，Condition对象也可以唤醒其他处于等待的线程。

Condition实例被绑定在一个Lock对象上，通过调用Lock对象的newCondition()方法即可获得。  
Condition类提供了如下三个方法：
![multithreading+20210611204900](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611204900.png%2B2021-06-11-20-49-01)

### BlockingQueue
BlockingQueue是Queue的子接口，它主要用来做线程同步工具。当生产者线程试图向BlockingQueue中放入元素时，如果该队列已满，则该线程被阻塞；当消费者线程试图从BlockingQueue中取出元素时，如果该队列已空，则该线程被阻塞。程序的两个线程通过交替向BlockingQueue中放入元素、取出元素，即可很好地控制线程的通信。  
BlockingQueue提供如下两个支持阻塞的方法：
![multithreading+20210611205233](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611205233.png%2B2021-06-11-20-52-33)

![multithreading+20210611205434](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611205434.png%2B2021-06-11-20-54-36)

## 线程组 和 未处理的异常
Java使用ThreadGroup来表示线程组，它可以对一批线程进行分配管理。

用户创建的所有线程都属于指定线程组，如果没有显式指定线程属于哪个线程组，则该线程属于默认线程组。在默认情况下，子线程和创建它的父线程初一同一个线程组内。一旦某个线程加入了指定线程组之后，该线程将一直属于该线程组，直到该线程死亡，线程运行中途不能改变它所属的线程组。

![multithreading+20210611215920](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611215920.png%2B2021-06-11-21-59-21)

![multithreading+20210611220002](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611220002.png%2B2021-06-11-22-00-03)

![multithreading+20210611220248](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611220248.png%2B2021-06-11-22-02-50)

## 线程池
系统启动一个新线程的成本是比较高的，当程序中需要创建大量生存期很短暂的线程时，应该考虑实现线程池。除此之外，使用线程池可以有效控制系统中并发线程的数量。

线程池在系统启动时即创建大量空闲的线程，程序讲一个Runnable对象或者Callable对象传给线程池，线程池就会启动一个空闲的线程来执行它们的run()或call()方法，当run()或call()方法执行结束后，该线程并不会死亡，而是再次返回线程池中成为空闲状态，等待执行下一个Runnable对象的run()或call()方法。

### 创建线程池
Java使用Executors工厂类来产生线程池，该工程类包含如下几个静态工厂方法来创建线程池：
![multithreading+20210611221449](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611221449.png%2B2021-06-11-22-14-50)

![multithreading+20210611222258](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611222258.png%2B2021-06-11-22-22-59)

### 提交任务
ExecutorService代表尽快执行线程的线程池，程序只要将一个Runnable对象或Callable对象提交给该线程池，该线程池就会尽快执行该任务。  
ExecutorService里提供如下三个方法：
* Future<?> submit(Runnable task)：将一个Runnable对象提交给指定的线程池，线程池将在有空闲线程时执行Runnable对象代表的任务。其中Future对象代表Runnable任务的返回值——但run()方法没有返回值，所以Future对象将在run()方法执行结束后返回null。但可以调用Future的isDone()、isCancelled()方法来获得Runnable对象的执行状态。
![multithreading+20210611223455](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223455.png%2B2021-06-11-22-34-55)

![multithreading+20210611223516](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223516.png%2B2021-06-11-22-35-17)

### 关闭线程池
![multithreading+20210611223549](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223549.png%2B2021-06-11-22-35-50)

### 完整步骤
![multithreading+20210611223611](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223611.png%2B2021-06-11-22-36-11)

### ForkJoinPool
![multithreading+20210611223749](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223749.png%2B2021-06-11-22-37-51)

## 线程相关类

### ThreadLocal类
![multithreading+20210611223905](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223905.png%2B2021-06-11-22-39-06)

### 包装不安全的集合类
![multithreading+20210611223957](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611223957.png%2B2021-06-11-22-39-59)

### 线程安全的集合类
![multithreading+20210611224040](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611224040.png%2B2021-06-11-22-40-40)

![multithreading+20210611224255](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611224255.png%2B2021-06-11-22-42-56)

![multithreading+20210611224312](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611224312.png%2B2021-06-11-22-43-14)

![multithreading+20210611224324](https://raw.githubusercontent.com/loli0con/picgo/master/images/multithreading%2B20210611224324.png%2B2021-06-11-22-43-25)

## 后记
这篇笔记不能看，太拉垮了，后面还要做很多修改

## 参考
https://zhuanlan.zhihu.com/p/133275094  
https://www.zhihu.com/question/27406575