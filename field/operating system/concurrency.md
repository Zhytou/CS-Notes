# 并发模型

- [并发模型](#并发模型)
  - [进程 Process](#进程-process)
    - [进程API Process API](#进程api-process-api)
    - [进程的地址空间 Process Address Space](#进程的地址空间-process-address-space)
    - [进程间通信 IPC](#进程间通信-ipc)
    - [进程调度 Process Scheduling](#进程调度-process-scheduling)
    - [进程与任务管理 Process \& Job Control](#进程与任务管理-process--job-control)
    - [特殊的进程](#特殊的进程)
  - [线程 Thread](#线程-thread)
    - [POSIX线程API](#posix线程api)
    - [线程同步](#线程同步)
  - [协程 Coroutine](#协程-coroutine)
  - [基于事件的并发模型](#基于事件的并发模型)
    - [事件循环](#事件循环)
  - [参考 Reference](#参考-reference)

进程、线程和协程这些概念的提出其实都或多或少为了允许计算机在有限的计算资源下能够同时执行多个任务，即并发地运行程序。也就是说，进程、线程、协程都是操作系统中重要的并发模型。

![图1 进程 vs 线程 vs 协程](https://blog.kennycoder.io/2020/05/16/%E9%80%B2%E7%A8%8B-Process-%E3%80%81%E7%B7%9A%E7%A8%8B-Thread-%E3%80%81%E5%8D%94%E7%A8%8B-Coroutine-%E7%9A%84%E6%A6%82%E5%BF%B5%E8%AC%9B%E8%A7%A3/cover.png)

比如，进程的提出就是为了互不干扰地为多个用户提供服务。从用户的角度来看，多进程通过CPU调度技术（CPU Scheduling）和进程地址空间（Process Address Space）使得计算和存储资源都得到了隔离和抽象。换句话说，进程就是CPU的虚拟化，地址空间就是内存的虚拟化。这也是贯穿操作系统的思想，即虚拟化，包括后续的文件/目录（块设备的虚拟化）也使用了这一思想。而线程则是进一步达成细粒度的并发执行，更高效地利用CPU。最后，协程则在线程的基础之上，再一次减少了上下文切换的开销。

**并发 vs 并行**：

并发是指任务的时间上的重叠执行。它可以容忍只有一个CPU资源，因为CPU可以分时间片交替执行，从而模拟出任务执行重叠的效果。而并行则是指一个或多个任务同时在多个CPU上真正同时执行。

## 进程 Process

**进程与程序**：

进程和程序组成了一对相依的概念。一个运行中的程序被称为一个进程；反过来讲，进程就是程序的一个实例。更准确的说，一个正在执行、且拥有独立的内存空间和资源的程序就是进程。

**进程与状态机**：

![图2 进程的状态](https://pic3.zhimg.com/80/v2-4a98dc88b067603584f3c0d6a688b4f6_1440w.webp)

尽管我们说一个正在执行的程序是进程，但进程作为一个状态机，不光有正在执行（Running）的状态，它还具有如图2中所展示的其他状态，包括：

- 新建状态（Created/New）：等待资源分配
- 就绪状态（Ready）：等待CPU资源
- 阻塞状态（Waiting）：等待其他资源，如I/O等
- 中止状态（Terminated）：进程执行结束

值得注意的是，进程的状态可以分成五态机和三态机，此处提到的是五态机。此外，还需注意的是，一旦进程因I/O进入等待状态，只能先转为就绪再转为运行状态。换句话说，就是只有就绪和运行状态可以互相转换。

**进程控制块**：

进程控制块（Process Control Block，PCB）是操作系统为每个进程维护在内核中的一个数据结构，保存着进程的相关信息。

**进程的标识**：

进程的描述符不是一个简单的符号，而是由多种信息（进程优先级、状态、虚拟地址以及访问权限等等）组成的一个复杂数据结构。而被保存在其中的进程ID（Process ID）则可以认为是进程在操作系统中的唯一标识。

### 进程API Process API

**fork()**：做一份状态机的完整复制（内存、寄存器现场）。

`pid_t fork(void)`

在fork函数返回后，会有两个进程同时运行，其中一个是父进程，另一个是新创建的子进程。

在子进程创建成功后，它会复制父进程的代码和数据段，并且开始执行从fork函数返回的位置。也就是说，子进程会从fork函数返回的地方开始执行，而不是从程序的起始位置开始执行。

需要注意的是，在子进程中，fork函数的返回值是0，而在父进程中，fork函数的返回值是新创建的子进程的进程ID。因此，在父进程中可以通过判断fork函数的返回值来区分当前进程是父进程还是子进程。

**execve()**：将当前运行的状态机重置成另一个程序的初始状态。

`int execve(const char *pathname, char *const argv[], char *const envp[]);`

- pathname：指定要执行的程序的路径，可以是绝对路径或者相对路径。如果路径中不包含目录分隔符 /，则会从环境变量 PATH 中查找可执行文件。
- argv：一个以 NULL 结尾的字符串数组，用于指定新程序的命令行参数。第一个元素通常是程序的名称，后面的元素是程序的参数。注意，第一个元素是程序的名称，而非路径名。
- envp：一个以 NULL 结尾的字符串数组，用于指定新程序的环境变量。每个字符串都是一个形如 key=value 的键值对，表示一个环境变量的值。

execve函数执行成功时，不会返回到调用进程，而是直接将当前进程替换为新程序的进程。如果执行失败，则会返回 -1，并设置相应的错误码。

### 进程的地址空间 Process Address Space

**提出背景**：

为了多个程序能同时驻留在内存且互不干扰地运行，操作系统使用虚拟内存技术提供了一种地址空间的机制（Address Space）来保证隔离性。通过地址空间，操作系统向上层应用屏蔽了存储细节，完成了对物理内存的抽象。换句话说，进程空间就是程序能够看到的内存。

**空间组成**：

![图3 进程地址空间](../img/process_address.png)

一个进程的地址空间包含了程序的所有运行状态，如图3进程地址空间所示。它总体上可以分成两大部分，即用户空间和内核空间。其中，前者也就是实现进程隔离性的核心。而后者虽然是共享的（映射了同一段物理内存），但仅仅只有当进程需要执行系统调用，并切换到内核态之后才会真正访问。（有关内核空间这部分的相关知识会在进程调度章节中更详细说明。）

对于用户空间部分，它一般由以下几个部分组成：

- 代码区(存储程序代码)
- 全局数据区(存储全局变量和静态变量)
- 栈区(存储函数调用时创建的局部变量和参数)
- 堆区(存储动态分配的内存)

而对于内核空间，它则一般包含：

- 内核代码和数据
- 系统调用表
- 设备驱动
- 中断处理程序
- 进程描述符
- 缓冲区
- 设备寄存器虚拟映射表

### 进程间通信 IPC

在多进程程序中，各个程序的协作通常依靠`进程间通讯IPC(Interprocess Communication)`来完成。

**分类 Classification**：

Linux系统中，IPC被分成三大类：

- 基于数据的IPC方法
  - 匿名 & FIFO 管道
  - 消息队列
  - 共享内存
  - 套接字
- 基于信号的IPC方法
  - IPC中唯一一种异步的通讯方法
- 基于同步的IPC方法
  - 信号量
  - 互斥量（一种特殊的信号量，只有锁定和非锁定两种状态）

**管道 Pipeline**：

管道（Pipeline）是一种用于在两个进程之间传递数据的通信机制。以匿名管道为例，它本质上是内核内存中一段内存缓存。生产者从一个管道一段中写入数据，消费者从另一端读出数据。由于其单向的特性，它属于一种半双工通信。比如，下面这段代码就是使用匿名管道进行子父进程通信的例子。

```c++
#include<unistd.h>

int main(int argc, char *argv[]) {
  int fd[2];
  if (pipe(fd) == -1) {
    // 创建管道失败
  }
  int pid = fork();
  if (pid == -1) {
    // 子进程创建失败
  }else if (pid == 0) {
    // 子进程
    close(pipefd[1]); // 关闭写端
    read(pipefd[0], buffer, sizeof(buffer)); // 从管道中读取数据
    printf("Child process received: %s\n", buffer);
    close(pipefd[0]);
  } else {
    // 父进程
    close(pipefd[0]); // 关闭读端
    write(pipefd[1], "Hello, child\n", 14); // 向管道中写入数据
    close(pipefd[1]);
  }
}
```

不过，匿名管道无名字的特点决定了它只能被有亲属关系的进程使用。因为仅有亲属关系的进程能够共享文件句柄。因此，工程师提出了有名管道，它也被称为FIFO管道。不同于匿名管道，它是一种特殊文件。即使创建其的进程退出后，也能在文件系统中找到它进行二次读取或写入。它的创建函数原型如下：

```c++
int mkfifo(const char *pathname, mode_t mode);
/*
创建有名管道，指令 man 3 mkfifo 查看详情
头文件： #include <sys/types.h>
        #include <sys/stat.h>
pathname：创建管道文件的路径，带管道文件名
mode：文件权限。与open函数中的mode一样，需要与umask做相应运算去掉某些权限
返回值：
    成功返回0
    失败返回-1
*/
```

一旦使用mkfifo创建了一个FIFO管道，就可以使用open打开它进行写入或读取，整体使用方法与匿名管道类似。总结一下二者的异同：

- 二者都是半双工通信，本质都是一块缓存。生产者和消费者分别进行写入和读取，从而达成通信。
- 匿名管道只允许有亲属关系的进程使用，而有名管道无限制。

**共享内存**：

共享内存（Shared Memory）也是一种进程间通信的机制。具体来说，它将同一物理地址映射到不同进程的虚拟地址中，从而允许某内存被多个进程同时访问和修改。在Linux系统中，共享内存通常可以通过SystemV API和POSIX API来实现。其中，POSIX API主要通过以下函数实现：

- `int shm_open(const char *name, int oflag, mode_t mode);`: 创建或打开共享内存对象。
- `int shm_unlink(const char *name);`:删除共享内存对象。
- `void *mmap(void addr[.length], size_t length, int prot, int flags, int fd, off_t offset);`: 将共享内存映射到进程的地址空间。
- `int munmap(void addr[.length], size_t length);`:共享内存从进程的地址空间分离。

下面给出POSIX API实现共享内存的例子：

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <sys/stat.h>
#include <sys/mman.h>
#include <unistd.h>
 
int producer() {
  /* the size (in bytes) of shared memory object */
  const int SIZE = 4096;

  /* name of the shared memory object */
  const char* name = "OS";

  /* strings written to shared memory */
  const char* message_0 = "Hello";
  const char* message_1 = "World!";

  /* shared memory file descriptor */
  int shm_fd;

  /* pointer to shared memory object */
  void* ptr;

  /* create the shared memory object */
  shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);

  /* configure the size of the shared memory object */
  ftruncate(shm_fd, SIZE);

  /* memory map the shared memory object */
  ptr = mmap(0, SIZE, PROT_WRITE, MAP_SHARED, shm_fd, 0);

  /* write to the shared memory object */
  sprintf(ptr, "%s", message_0);
  ptr += strlen(message_0);
  sprintf(ptr, "%s", message_1);
  ptr += strlen(message_1);
  return 0;
}

void consumer() {
  /* the size (in bytes) of shared memory object */
  const int SIZE = 4096;

  /* name of the shared memory object */
  const char* name = "OS";

  /* shared memory file descriptor */
  int shm_fd;

  /* pointer to shared memory object */
  void* ptr;

  /* open the shared memory object */
  shm_fd = shm_open(name, O_RDONLY, 0666);

  /* memory map the shared memory object */
  ptr = mmap(0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);

  /* read from the shared memory object */
  printf("%s", (char*)ptr);

  /* remove the shared memory object */
  shm_unlink(name);
  return 0;
}
```

**套接字 Socket**：

套接字（Socket）是一种Unix系统中进程间通信的机制。它提供了一组标准函数接口，隐藏了器底层实现协议，从而统一了不同的网络编程模型（流式和数据包式、面向连接和无连接、TCP和UDP）。套接字本质上是一个文件描述符（即字节流），允许连接双方向其中写入或读取数据，从而达到通信的目的。它常见的API包括：

- connect()：客户端尝试连接某服务器
- accept()：服务器接收某客户端请求
- send()/recv()：发送和接收数据
- bind()：服务器让套接字绑定到某端口
- listen()：服务器让套接字进入被动模式，监听bind端口的连接请求（若未提前bind，则监听任意一个可用端口）
- close()：关闭连接

一个常见的客户端使用套接字向服务器发送信息的API调用顺序如下：

- 首先，服务器创建一个套接字，并使用bind将其绑定到某IP和端口，接着使用listen使其监听该端口的连接请求。
- 接着，客户端创建一个套接字，并使用connect尝试连接服务器。成功后，使用send向其发送请求。
- 此时，服务器介绍到客户端请求，使用accept得到一个新套接字，先使用recv读取请求，接着使用send发送回复。
- 接着，客户端使用recv接收到回复，使用close关闭连接。
- 最后，服务器使用close关闭连接。

关于其更详细的介绍可以查看[这篇笔记](https://github.com/Zhytou/CS-Notes/blob/main/field/computer%20networking/socket.md)。

### 进程调度 Process Scheduling

尽管我们一再指出进程实现了CPU的虚拟化，即以某种方式让多个任务在某时段共享CPU，使它们看起来像是并发执行。这背后的基本思想就是时分共享（Time Sharing）CPU，但实现这种机制的前提则是进程能够正确切换。换句话说，虚拟化CPU的前提是操作系统能够有效的运行进程，并保留部分CPU控制权以在恰当的时候进行进程的上下文切换。

因此，我会在这一部分讲一讲进程切换的底层机制和常见调度策略。

**CPU虚拟化的前提**：

进程在操作系统中是处于一种受限制直接执行（Limited Direct Execution）的模式之中的。其中，直接执行十分好理解，即程序直接作用于CPU。而受限制则是指进程默认处于用户模式（User Mode），无法访问一些硬件资源。只有当其使用陷入（Trap）内核指令切换到内核模式（Kernel Mode），才能访问机器的所有硬件资源。此外，操作系统还提供了一种陷阱返回（Return From Trap）指令来支持内核模式到用户模式的切换。

尽管操作系统通过用户和内核两种模式，使得进程在需要一些特殊硬件资源时，必须将CPU的控制权返还给操作系统，但这仅仅只能覆盖部分情况，当进程不需要访问硬件资源时，操作系统又该怎么做呢？答案是，利用时钟中断（Timer Interrupt）。当产生一个时钟中断时，当前正在运行的进程停止，操作系统中预先配置的中断处理程序（Interrupt Handler）会运行。此时，操作系统重新获得CPU的控制权。这也就是操作系统进程切换的底层机制。

**上下文切换**：

上下文切换（Context Switch）在概念上很简单：操作系统要做的就是为当前正在执行的进程保存一些寄存器的值，并为即将执行的进程恢复一些寄存器的值。这样一来，操作系统就可以确保最后执行从陷阱返回指令时，不是返回到之前运行的进程，而是继续执行另一个进程。C语言中也提供了`setjmp`和`longjmp`这一对函数来实现上下文切换。

**进程调度的策略**：

- 先到先服务（First Come First Served）：按照请求的顺序进行调度。
- 最短任务优先（Shortest Job First）：按估计运行时间最短运行。但存在一直收到短时间任务，长时间任务永远得不到调度的情况。
- 最短剩余时间有限（Shortest Remaining Time Next）：最短任务优先的改进版本，按剩余运行时间的顺序进行调度。
- 轮转（Round-Robin）
  - 时间片轮转：所有就绪任务按注册时间排序，每次调度时，将CPU分配给队首任务，使其执行一个固定大小的时间片。算法效率和时间片的大小有很大关系。如果时间片太小，会导致进程切换得太频繁，在进程切换上就会花过多时间。而如果时间片过长，那么实时性就不能得到保证。
  - 优先级调度：为每个进程分配一个优先级，按优先级进行调度。

### 进程与任务管理 Process & Job Control

在终端中，我们能够使用Shell执行一些命令，同时也能查看执行情况并且中止或暂停执行，即：进行任务管理。

比如，我们可以使用Ctrl+C或Ctrl+Z快捷键中止或暂停（在后台挂起）一个正在运行的进程，也能使用jobs、fg、bg等Shell内置命令查看所有进程、在前台恢复某个暂停进程、在后台恢复某个暂停进程。那么其背后的原理是什么呢？如果我fork了一份计算任务，使用Ctrl+C会杀死哪个任务呢？为什么有些时候会出现无法结束进程的情况呢？

**信号 Signal**：

简单来说，终端依靠信号完成任务管理。信号（Signal）是一种进程间的通信方式。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。信号本质上是软件模拟的中断请求。常见的信号包括：

- SIGINT:程序终止信号，通常通过Ctrl+C组合键发送。
- SIGTERM:另一种更优雅、更通用的程序终止信号，可以使用`kill`命令发出。
- SIGSTOP:程序暂停信号。
- SIGTSTP:另一种程序暂停信号，通常通过Ctrl+Z组合键发送（其中的T代表Terminal，即终端版本的暂停信号）。

此外，大家熟悉的SIGSEGV也是一种信号，它通常发生在程序出现段错误时，即访问到无效地址（野指针）。

因此，当我们在终端中输入Ctrl+C时，它会发送一个SIGINT信号到它连接会话（Session）中的前台进程组（Foreground Process Group）。而前台进程在收到该信号后则会中止。那么，什么是所谓的会话呢？终端中的会话概念和计算机网络中的会话概念有什么关系呢？

**会话 Session**：

会话（Session）广义上是指两个程序建立连接并发送交互的状态。在计算机的世界中，会话这个概念十分常见。比如，在计算机网络领域，会话是一种在服务器端保存数据的机制。它常常和插件（Cookie）、令牌（Token）等机制比较，具体介绍可以查看我的[笔记](https://github.com/Zhytou/CS-Notes/blob/main/field/computer%20networking/application%20layer/cookie%20%26%20session.md)。

本文想讨论的主要是终端会话（Terminal Session）。它往往由多个进程组（Process Group）构成，并且一定由一个控制终端（Controlling Terminal）连接着（或者说，会话就是由这个终端开启的），具体如下图。

![session & terminal & process group](https://jyywiki.cn/pages/OS/img/tty-session.png)

除此之外，要补充的是所有fork出的进程都会继承进程组号。这也就能回答本章开头的问题了——如果我fork了一份计算任务，使用Ctrl+C会杀死哪个任务呢？答案是，所有fork出的进程都中止。

**任务管理的方法**：

在了解了基本的任务管理原理之后，我想再补充一下常见的任务管理方法。

- 快捷键：Ctrl+C或Ctrl+Z快捷键中止或暂停（在后台挂起）。
- Shell命令：
  - jobs查看所有进程。
  - fg在前台恢复某个暂停进程。
  - bg在后台恢复某个暂停进程。
  - daemonize将一个给定的进程变换为守护进程模式运行。
  - nohup让命令在后台持续运行,即使登出登录会话或关闭终端也不会被终止。
  - crontab配置定时任务。
- 系统调用：
  - setsid()：设置进程为新的会话领导者。
  - setpgid(pgid_t)：设置进程组号，让进程加入某个进程组。
  - daemon()：创建守护进程。

此处再补充一下Unix/Linux通常涉及到以下几类ID：

- PID:进程ID，唯一标识一个进程。
- TID:线程ID，唯一标识一个进程内的线程。
- PPID:父进程ID，标识一个进程是由哪个父进程fork出来的。
- PGRP:进程组ID，标识一个后台作业中的进程集合。
- SID:会话ID，标识一个登录会话下的所有进程组。

### 特殊的进程

**Init进程**：

Init进程是Unix/Lunix系统中用来产生其它所有进程的程序。

**守护进程 Daemon Process**：

守护进程（Daemon Process）指的是运行在后台并为系统和用户提供服务的进程。

补充：可以使用nohup命令将进程转为后台、daemonize命令将进程守护进程化、编写Systemd脚本并使用该系统启动和管理运行级进程。（Systemd是Linux系统的第一个全功能的初始化系统和服务管理器。）

**孤儿进程 Orphan Process**：

孤儿进程（Orphan Process）指的是其父进程已退出，但自身仍在运行的进程。一般来说，操作系统会将这些孤儿进程重新纳入Init进程的控制之下。

**僵尸进程 Zombie Process**：

僵尸进程（Zombie Process）是指的是自身已经运行结束，但其父进程还未调用wait()系统调用回收其资源的进程。

尽管它自身已经结束执行，但进程表中还保存有它的系统资源情况。所以这些进程呈现出的状态为"僵死"状态，不能被再次使用，也不能被系统自动回收。

相比之下，孤儿进程对系统影响较小，僵尸进程如果过多会占用系统资源影响性能。

## 线程 Thread

线程（Thread）是进程中的一个执行流程。换句话说，进程是操作系统管理的基本单位，它可以独立运行且具有自己的地址空间和资源。而线程是存在于进程中的一个执行实体，它是在进程中完成某个任务或代码片段的一个单元。

更具体的说，进程是资源分配的基本单位，而线程则是CPU调度的最小单位。当它们都发生上下文切换时，后者只需要重置CPU中的寄存器，而前者还需要更换地址空间和申请其他硬件资源。

**线程控制块**：

线程控制块（Thread Control Block，TCB）是操作系统为每个线程维护在内核中的一个数据结构，保存着线程的相关信息。

**线程的标识**：

和进程类似，线程也有属于自己的ID（Thread ID），但线程ID在系统范围内可以不是唯一的，它只在所属的进程范围内唯一。

**线程实现模型**：

线程的实现模型主要有3个，它们之间最大的区别就是与`内核调度实体KSE(Kernel Scheduling Entity)`的对应关系。顾名思义，内核调度实体就是可以被内核调度器调度的对象，也就是操作系统内核的最小调度单元。

- 用户级线程模型；用户级线程模型是由用户程序库提供的线程管理机制，内核不直接支持线程。在这种模型下，用户线程的创建、调度、同步、通信等操作都由用户程序库完成，内核只负责进程的管理和调度。用户级线程模型的优点是轻量级、灵活性高，但是缺点是无法利用多核处理器的优势，因为内核无法感知并发执行的用户线程。
- 内核级线程模型：内核级线程模型是由内核提供的线程管理机制，内核负责线程的创建、调度、同步、通信等操作。在这种模型下，每个线程都有对应的内核线程，内核可以感知并发执行的内核线程并进行调度。内核级线程模型的优点是能够充分利用多核处理器的优势，缺点是线程切换的开销较大，且线程同步和通信需要通过内核来完成，效率较低。
- 两级线程模型：混合线程模型是用户级线程模型和内核级线程模型的结合，既可以利用用户级线程模型的轻量级和灵活性，又可以利用内核级线程模型的多核处理器优势。在混合线程模型下，每个进程可以有多个用户线程，由用户程序库进行管理，同时每个用户线程对应一个内核线程，由内核进行调度。当一个用户线程被阻塞时，它对应的内核线程可以继续执行其他用户线程，从而提高系统的并发能力和吞吐量。

### POSIX线程API

> 可移植操作系统接口（Portable Operating System Interface，POSIX）是一种流行的操作系统应用程序接口、命令、工具和文件系统行为的规范。它的目标是定义一致和稳定的操作系统API，从而使程序在POSIX兼容的不同操作系统上运行。

POSIX Threads是POSIX定义的线程API。其常见接口包括：

- pthread_create() - 创建一个新线程
- pthread_join() - 等待线程终止
- pthread_exit() - 终止当前线程

此外，它还提供了一些互斥量、条件变量等线程同步机制的支持。

### 线程同步

- 共享内存
- 互斥量
- 条件变量

## 协程 Coroutine

协程（Coroutine）是一个轻量级的线程。和线程相比，协程的上下文切换更简单，它不需要切换到内核态依靠操作系统切换，而完全在用户态使用一套自己的机制实现上下文切换。Golang语言中的Gorountine就是一种典型的协程实现。有关Gorountine背后的原理，可以阅读[这些内容](https://www.zhihu.com/question/20862617)。此外，南大的操作系统课程也提供了一个非常有意思的实验[NJU OS M2: 协程库 (libco)](https://jyywiki.cn/OS/2022/labs/M2)，有兴趣的朋友也可以试着去完成一下。

## 基于事件的并发模型

### 事件循环

## 参考 Reference

- [NJU-OS 操作系统上的进程](https://jyywiki.cn/OS/2022/slides/11.slides#/)
- [Operating Systems: Three Easy Pieces：Address Spaces](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/13.pdf)
- [CSAPP: Computer System A Programmer's Perspective 第十二章 并发编程](https://hansimov.gitbook.io/csapp/part3/ch12-concurrent-programming)
- [Github Cyc2018/CS-Notes](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%20-%20%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86.md)
- [Process , Thread and Coroutines](https://www.linkedin.com/pulse/process-thread-coroutines-amit-nadiger/)
