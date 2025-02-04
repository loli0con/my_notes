# UNIX进程API

UNIX系统通过一对系统调用fork()和exec()来创建新进程。进程可以通过第三个系统调用wait()来等待其创建的子进程执行完成。

## fork()系统调用

系统调用fork()用于创建新进程：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// p1.c
int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {         // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {              // parent goes down this path (main)
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());
    }
    return 0;
}
```

运行这段程序（p1.c），将看到如下输出：

```
prompt> ./p1
hello world (pid:29146)
hello, I am parent of 29147 (pid:29146)
hello, I am child (pid:29147)
```

当p1.c刚开始运行时，进程输出一条hello world信息，以及自己的进程描述符（process identifier，PID）。该进程的PID是29146。在UNIX系统中，如果要操作某个进程（如终止进程），就要通过PID来指明。

进程调用了fork()系统调用，这是操作系统提供的创建新进程的方法。新创建的进程几乎与调用进程完全一样，对操作系统来说，这时看起来有两个完全一样的p1程序在运行，并都从fork()系统调用中返回。新创建的进程称为子进程（child），原来的进程称为父进程（parent）。子进程不会从main()函数开始执行（因此hello world信息只输出了一次），而是直接从fork()系统调用返回。

子进程并不是完全拷贝了父进程。具体来说，它拥有自己的地址空间（即拥有自己的私有内存）、寄存器、程序计数器等，它从fork()返回的值也和父进程不同。父进程获得的返回值是新创建子进程的PID，而子进程获得的返回值是0。

假设在单个CPU的系统上运行，那么子进程或父进程在此时都有可能运行，CPU调度程序（scheduler）决定时了某个刻哪个进程被执行。在上面的例子中，父进程先运行并输出信息。在其他情况下，子进程可能先运行，会有下面的输出结果：

```
prompt> ./p1
hello world (pid:29146)
hello, I am child (pid:29147)
hello, I am parent of 29147 (pid:29146)
```

## wait()系统调用

有时候父进程需要等待子进程执行完毕，这项任务由wait()系统调用（或者更完整的兄弟接口waitpid()）完成：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

// p2.c
int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {         // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {              // parent goes down this path (main)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
    }
    return 0;
}
```

在上面的例子中，父进程调用wait()，延迟自己的执行，直到子进程执行完毕。当子进程结束时，wait()才返回父进程，因此输出结果也变得确定了——子进程总是先输出结果：

```
prompt> ./p2
hello world (pid:29266)
hello, I am child (pid:29267)
hello, I am parent of 29267 (wc:29267) (pid:29266)
```

* 如果子进程先运行，那么子进程先于父进程输出结果。
* 如果父进程先运行，那么它会马上调用wait()，该系统调用会在子进程运行结束后才返回。因此，即使父进程先运行，它也会等待子进程运行完毕，然后wait()返回，接着父进程才输出自己的信息。

## exec()系统调用

exec()系统调用可以让子进程执行与父进程不同的程序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

// p3.c
int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {         // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc");   // program: "wc" (word count)
        myargs[1] = strdup("p3.c"); // argument: file to count
        myargs[2] = NULL;           // marks end of array
        execvp(myargs[0], myargs);  // runs word count
        printf("this shouldn't print out");
    } else {              // parent goes down this path (main)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
    }
    return 0;
}
```

在这个例子中，子进程调用execvp()来运行字符计数程序wc。实际上，它针对源代码文件p3.c运行wc，从而告诉我们该文件有多少行、多少单词，以及多少字节：

```
prompt> ./p3 
hello world (pid:29383)  
hello, I am child (pid:29384) 
      29     107   1030 p3.c 
hello, I am parent of 29384 (wc:29384) (pid:29383)
```

给定可执行程序的名称（如wc）及需要的参数（如p3.c）后，exec()会从可执行程序中加载代码和静态数据，并用它覆写自己的代码段（以及静态数据），堆、栈及其他内存空间也会被重新初始化。然后操作系统就执行该程序，将参数通过argv传递给该进程。因此，它并没有创建新进程，而是直接将当前运行的程序（以前的p3）替换为不同的运行程序（wc）。子进程执行exec()之后，几乎就像p3.c从未运行过一样。对exec()的成功调用永远不会返回。

## 为什么这样设计API

这种分离fork()及exec()的做法谁构建UNIX shell的时候非常有用，因为这给了shell在fork之后、exec之前运行代码的机会，这些代码可以在运行新程序前改变环境，从而让一系列功能很容易实现。

shell也是一个用户程序，它首先显示一个提示符（prompt），然后等待用户输入。你可以向它输入一个命令（一个可执行程序的名称及需要的参数），大多数情况下，shell可以在文件系统中找到这个可执行程序，调用fork()创建新进程，并调用exec()的某个变体来执行这个可执行程序，调用wait()等待该命令完成。子进程执行结束后，shell从wait()返回并再次输出一个提示符，等待用户输入下一条命令。

fork()和exec()的分离，让shell可以方便地实现很多有用的功能。比如：`prompt> wc p3.c > newfile.txt`。wc的输出结果被重定向（redirect）到文件newfile.txt中（通过newfile.txt之前的大于号来指明重定向）。shell实现结果重定向的方式也很简单，当完成子进程的创建后，shell在调用exec()之前先关闭了标准输出（standard output），打开了文件newfile.txt。这样，即将运行的程序wc的输出结果就被发送到该文件，而不是打印在屏幕上。

下面展示了重定向的一个程序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>

# p4.c
int main(int argc, char *argv[])
{
    int rc = fork();
    if (rc < 0) {         // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child: redirect standard output to a file
        close(STDOUT_FILENO);
        open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);
        
        // now exec "wc"...
        char *myargs[3];
        myargs[0] = strdup("wc");     // program: "wc" (word count)
        myargs[1] = strdup("p4.c");   // argument: file to coun
        tmyargs[2] = NULL;            // marks end of array
        execvp(myargs[0], myargs);    // runs word count
    } else {              // parent goes down this path (main)
        int wc = wait(NULL);
    }
    return 0;
}
```

重定向的工作原理，是基于对操作系统管理文件描述符方式的假设。具体来说，UNIX系统从0开始寻找可以使用的文件描述符。谁这个例子中，STDOUT_FILENO将成为第一个可用的文件描述符，因此在open()被调用时，得到赋值。然后子进程向标准输出文件描述符的写入（例如通过printf()这样的函数），都会被透明地转向新打开的文件，而不是屏幕。下面是运行p4.c的结果：

```
prompt> ./p4
prompt> cat p4.output
      32     109    846 p4.c
```

p4调用了fork来创建新的子进程，之后调用execvp()来执行wc。屏幕上没有看到输出，是由于结果被重定向到文件p4.output。

UNIX管道也是用类似的方式实现的，但用的是pipe()系统调用。在这种情况下，一个进程的输出被链接到了一个内核管道（pipe）上（队列），另一个进程的输入也被连接到了同一个管道上。因此，前一个进程的输出无缝地作为后一个进程的输入，许多命令可以用这种方式串联在一起，共同完成某项任务。比如通过将grep、wc命令用管道连接可以完成从一个文件中查找某个词，并统计其出现次数的功能：`grep -o foo file | wc -l`。

## 其他API

除了上面提到的fork()、exec()和wait()之外，在UNIX中还有其他许多与进程交互的方式。比如可以通过kill()系统调用向进程发送信号（signal），包括要求进程睡眠、终止或其他有用的指令。实际上，整个信号子系统提供了一套丰富的向进程传递外部事件的途径，包括接受和执行这些信号。

此外还有许多非常有用的命令行工具，比如通过ps命令来查看当前在运行的进程，工具top展示当前系统中进程消耗CPU或其他资源的情况。