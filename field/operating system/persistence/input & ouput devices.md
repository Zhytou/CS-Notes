# I/O 设备

- [I/O 设备](#io-设备)
  - [标准I/O设备](#标准io设备)
  - [设备控制器与驱动](#设备控制器与驱动)
  - [轮询](#轮询)
    - [轮询的流程](#轮询的流程)
  - [中断](#中断)
    - [信号和软中断](#信号和软中断)
    - [中断的处理流程](#中断的处理流程)
  - [DMA](#dma)
  - [总线](#总线)
  - [I/O模型](#io模型)
    - [select](#select)
    - [poll](#poll)
    - [epoll](#epoll)
  - [参考](#参考)

I/O设备（Input/Output Device）即输入/输出设备，指的是可以与计算机交互的硬件设备。常见的I/O设备有：网卡、硬盘、键盘、鼠标、打印机等。一般来说，I/O设备可以分成字符设备（Character Device）、块设备（Block Device）和网络设备（Network Device）。其中，字符设备一般用于人机交互，比如：键盘、打印机、鼠标等。它们大多是以字符为单位发送和接收数据的。而块设备，一般用于持久化存储，比如：传统硬盘、固态硬盘、USB存储器等。它们大多是以块为单位进行传输的。

## 标准I/O设备

我们现在来看一个抽象概念的I/O设备，来帮助我们理解其架构和交互机制。

![图1 标准I/O设备](../../img/operating_system_io_device.png)

由上图可知，它往往包含两部分组成。第一部分是向上暴露的硬件接口（Hardware Interface），允许上层应用通过该接口来控制它的操作；第二部分是它的内部结构（Internal Structure），负责实现接口展示的功能。它的内部结构通常包含简单的主控芯片、一些通用内存、设备相关的特定芯片以及它ROM上的固件（Firmware），以实现其功能。

此外，硬件接口往往可以抽象和简化成三类寄存器：一个状态寄存器（Status Register），可以读取并查看设备的当前状态；一个命令寄存器（Command Register），用于通知设备执行某个具体任务；一个数据寄存器（Data Register），将数据传给设备或从设备接收数据。通过读写这些寄存器，系统可以控制设备的行为。

## 设备控制器与驱动

在真实的计算机系统中，设备控制器（Device Controller）承担了图1中硬件接口的职能，它们为操作系统屏蔽了设备之间的差异。尽管如此，但每种设备的控制器的寄存器、缓冲区等使用模式都是不同的，所以为了屏蔽设备控制器的差异，工程师们引入了设备驱动程序（Device Driver）。

值得注意的是，设备控制器不属于操作系统范畴，它是属于硬件，而设备驱动程序属于操作系统的一部分，操作系统的内核代码可以像本地调用代码一样使用设备驱动程序的接口，而设备驱动程序是面向设备控制器的代码，它发出操控设备控制器的指令后，才可以操作设备控制器。

![图2 设备控制器与设备驱动](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA/%E9%A9%B1%E5%8A%A8%E7%A8%8B%E5%BA%8F.png)

## 轮询

轮询（Polling）是一种系统与I/O设备交互机制。它描述的是一种系统反复访问设备，来第一时间得到设备就绪信息的机制。

### 轮询的流程

- 首先，操作系统通过反复读取图1中设备的状态寄存器，等待设备进入可以接收命令的就绪状态。
- 其次，操作系统下发数据到数据寄存器。例如，你可以想象如果这是一个磁盘，可能需要多次写入操作，将一个磁盘块传递给设备。如果主CPU参与数据移动，我们就称之为编程的I/O（Programmed I/O，PIO）。
- 接着，操作系统将命令写入命令寄存器；这样设备就知道数据已经准备好了，它应该开始执行命令。
- 最后，操作系统再次通过不断轮询设备，等待并判断设备是否执行完成命令，得到一个指示成功或失败的错误码。

## 中断

尽管轮询已经足够应付一些简单的I/O操作，但它会使CPU占有率过高。因此，工程师们提出了一种新机制——中断（Interrupt）来减少CPU开销。

**中断分类**：

一般来说，中断可以分为硬中断和软中断。其中，前者通常与外部事件相关并由硬件或外设产生，比如：时钟中断、I/O设备中断；而后者则是依靠CPU指令产生，比如：Intel CPU定义int等指令。实际上，系统调用就是使用软中断切换到内核态，进而执行相关操作的。

### 信号和软中断

信号（Signal）作为进程间通信（InterProcess Communication，IPC）的方式之一，它也常常被当作是一种对中断的软件模拟。那么信号和软中断的关系是什么呢？它是否是依靠中断实现的呢？

答案是，信号只是工作方式与中断类似，即都达成了异步通知的作用。此外，信号也不一定依赖中断实现，可能某些硬件平台使用了中断，有的没有。

> [知乎 软中断和信号是什么关系？](https://www.zhihu.com/question/33822078)

### 中断的处理流程

有了中断后，CPU不再需要不断轮询设备，而是向设备发出一个请求，然后就可以让对应进程睡眠，切换执行其他任务。当设备完成了自身操作，会抛出一个硬件中断，引发CPU跳转执行操作系统预先定义好的中断处理程序（Interrupt Handler）。

![图3 中断处理流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA/%E4%B8%AD%E6%96%AD%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.png)

> 中断处理程序是一小段操作系统代码，它会结束之前的请求并且唤醒等待I/O的进程继续执行。

值得注意的是，设备驱动程序初始化的时候，要先注册一个该设备的中断处理函数。

**中断的局限**：

尽管中断处理慢速的I/O请求有着巨大优势，但并非所有场景都时候适合使用中断处理I/O请求。比如，一个处理网络请求的服务器可能因为写入或读取每一个包中数据都发送中断，而无法处理用户的其他请求。

**中断 vs 异常**：

术语中断有时用作异常的同义词。区分两者的一种方法是，异常是导致指令的正常顺序执行被修改的事件（分支或跳转指令除外）。中断是不是由程序执行直接引起的异常。通常，处理器核心外部的硬件会发出中断信号，例如按下按钮。

## DMA

直接存取（Direct Memory Access）是一种可以协调完成内存和设备之间数据传输的设备，它是针对CPU参与IO的另一种改进方式。换句话说，DMA承担原本CPU的I/O职责，释放了CPU的控制权。

为了能够将数据传送给设备，操作系统会通过编程告诉DMA数据在内存的位置，要拷贝的大小以及要拷贝到哪个设备。在此之后，操作系统就可以处理其他请求了。当DMA的任务完成后，DMA 控制器会抛出一个中断来告诉操作系统自己已经完成数据传输。

## 总线

总线（Bus）是指计算机组件间规范化的交换数据的方式，即以一种通用的方式为各组件提供数据传送和控制逻辑。

**分类**：

- 数据总线（Data Bus）：在CPU与RAM之间来回传送需要处理或是需要储存的数据。
- 地址总线（Address Bus）：用来指定在RAM（Random Access Memory）之中储存的数据的地址。
- 控制总线（Control Bus）：将微处理器控制单元（Control Unit）的信号，传送到周边设备，一般常见的为USB Bus和1394 Bus。
- 扩展总线（Expansion Bus）：可连接扩展槽和电脑。
- 局部总线（Local Bus）：取代更高速数据传输的扩展总线。

**外部总线**：

外部总线，也叫I/O总线，指的就是用于连接CPU和外部I/O设备的总线。

## I/O模型

以上是I/O设备的一些主要概念，为了进一步方便程序员理解，操作系统/编程语言等都引入其相应的I/O模型来描述I/O设备在执行其操作时的行为特征。一般来说，一个I/O操作的流程如下：

- I/O调用：应用程序进程向操作系统内核发起调用。
- I/O执行：操作系统内核完成I/O操作。即内核先等待I/O设备准备好数据，再将其从内核缓冲区拷贝到用户进程缓冲区。

然而，操作系统可能针对不同I/O设备有不同处理机制，比如前面所介绍的轮询和中断机制。站在应用开发者的层面，这些机制包括：

- **阻塞I/O BIO**：即中断机制。应用程序被阻塞（睡眠），直至内核将数据拷贝到用户进程缓存。
- **非阻塞I/O NIO**：即轮询机制。应用程序不断检查I/O是否完成，直至不再接收到错误信息。
- **I/O复用**：操作系统提供select/poll/epoll函数。它们可以同时监控多个I/O的操作，任何一个返回内核数据就绪，就通知应用程序去执行。
- **信号驱动式I/O**：
- **异步I/O**：

具体来说，I/O复用技术的前提实际上是非阻塞I/O。因为其内部会会遍历集合中的每个文件描述符，判断其是否就绪。而关于就绪这一点，它表示的是套接字满足下述四个条件中任意一条即可。

### select

```c++
int select(int nfds, fd_set *restrict readfds, fd_set *restrict writefds, fd_set *restrict errorfds, struct timeval *restrict timeout);
```

select的函数原型如上所示。其中，nfds是用于指定要监视的文件描述符集合中最大的文件描述符值加1；readfds、writefds和errorfds三个指针分别指定要监视可读事件、可写事件和出错事件的文件描述符集合；tiemout用于指定等待时长。

**fd_set和timeval**：

此外，还需要补充的是fd_set类型以及timeval结构体的使用方法。前者实际是一个无符号整数，用其二进制形式中1的位置来表示已就绪文件符；而后者定义如下：

```c++
struct timeval {
  long tv_sec;  //seconds
  long tv_usec; // miscroseconds
}
```

一般来说，通过设置不同的tv_sec和tv_usec的值，可以达成以下三种效果：

- 永远等待下去：将timeout指针设为nullptr；
- 等待一段固定时间：设置timeval结构体中秒数和微秒数来指定等待时间；
- 根本不等待：将timeval结构体中秒数和微秒数均设为0.

**select的最大描述符数**：

select函数所能监视的文件描述符最大数目受`<sys/types.h>`中FD_SETSIZE宏大小的影响，目前一般是1024。如果想要增大这个最大值，除了修改这个宏之外，还需要重新编译内核。单单修改这个值是不够的。

**pselect函数**：

pselect函数由POSIX发明。它和select主要有两个区别：

- 使用timespec指定等待时间，最小可精确到纳秒；
- 增加了第六个参数：一个指向信号掩码的指针。

### poll

和select函数相比，poll函数除了没有最大监测文件符数量限制，几乎没有区别。二者都需要将待检测文件描述符从用户空间复制到内核空间。具体来说，poll函数原型如下：

```c++
int poll(struct pollfd *fds, unsigned int nfds, int timeout);
```

其中，fds是一个pollfd结构体类型的数组，表示需要监视的文件描述符；ndfs用于指定fds的大小，即文件描述符的数量；而timeout则用于指定等待毫秒数。同样的，poll函数的timeout同样可以达成三种效果，分别是：

- timeout为负数时，它会无限等待，直到第一个文件描述符就绪发生。
- timeout为零时，它会立刻轮询一次所有文件描述符，然后返回。
- timeout为正数时，它则会在指定时间内轮询所有文件描述符，直到超时或文件符就绪才返回。

**pollfd结构体**：

此外，不同于select函数通过传入三个位图readfds、writefds和errorfds来分别指定需要监视的读、写和错误事件，poll函数使用pollfd结构体来表示每个文件描述符需要监视的事件。该结构的定义如下：

```c++
 struct pollfd {
  int   fd;         /* file descriptor */
  short events;     /* requested events */
  short revents;    /* returned events */
};
```

其中，fd代表该文件描述符，而events和revents则分别表示需要监视的事件实际发生的事件。它们的有效值包括：

- POLLIN：有数据可读。
- POLLRDNORM：有普通数据可读。
- POLLRDBAND：有优先数据可读。
- POLLPRI：有紧迫数据可读。
- POLLOUT：写数据不会导致阻塞。
- POLLWRNORM：写普通数据不会导致阻塞。
- POLLWRBAND：写优先数据不会导致阻塞。
- POLLMSGSIGPOLL：消息可用。

值得注意的是，这些标志并不是互斥的：它们可能被同时设置，表示这个文件描述符的读取和写入操作都会正常返回而不阻塞。除此之外，revents还可能返回下列事件：

- POLLER：指定的文件描述符发生错误。
- POLLHUP：指定的文件描述符挂起事件。
- POLLNVAL：指定的文件描述符非法。

### epoll

epoll是对select和poll的改进，它使用红黑树管理文件描述符，且避免了将待监视文件描述符从用户空间复制到内核空间，同时还提供了边缘触发和水平触发两种模式供开发者选择。不光如此，epoll实际使用一组函数来帮助管理文件描述符。

**epoll_create函数**：

epoll_create函数用于创建一个epoll实例。它会返回一个引用该实例的文件描述符，以便其他epoll的API通过该文件描述符操作epoll实例。其函数原型如下。

```c++
int epoll_create(int size);
```

其中，参数size用于告诉内核该epoll实例大致会监听的事件数目，而不是类似select中最大监听数量加一。在Linux最新的一些内核版本的实现中，这个size参数甚至没有任何意义。此外，在使用完epoll后，必须调用close(fd)关闭对应的文件描述符，否则可能导致fd被耗尽。

**epoll_ctl函数**：

epoll_ctl函数用于向已经创建好的epoll实例中添加、修改或删除需要监视的文件描述符，其函数原型如下。

```c++
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

其中，epfd用于指定epoll实例（即epoll_create函数返回的文件描述符）；fd用于指定需要监视的文件描述符；event用于指定需要监听的事件；op用于指定要对fd执行的操作，一般包括：

- EPOLL_CTL_ADD：为fd添加一个监听事件。
- EPOLL_CTL_MOD：修改已经注册的fd的监听事件。
- EPOLL_CTL_DEL：删除fd的所有监听事件，这种情况下event参数没用。

此外，epoll_event结构体的定义如下：

```c++
typedef union epoll_data {
  void *ptr;
  int fd;
  __uint32_t u32;
  __uint64_t u64;
} epoll_data_t;

struct epoll_event {
  __uint32_t events; /* Epoll events */
  epoll_data_t data; /* User data variable */
};
```

**epoll_wait函数**：

epoll_wait函数是epoll模型最核心的函数，功能相当于select和poll。其原型如下：

```c++
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

其中，epfd即epoll_create返回的文件描述符，指向一个epoll实例；events是一个数组，保存就绪状态的文件描述符，其空间由调用者负责申请；maxevents用于指定最多可保存的就绪文件描述符的大小，即events的最大数量；
timeout类似于select中的timeout，它同样能达成三种效果：

- timeout为负数时，它会无限等待，直到第一个文件描述符就绪发生。
- timeout为零时，它会立刻轮询一次所有文件描述符，然后返回。
- timeout为正数时，它则会在指定时间内轮询所有文件描述符，直到超时或文件符就绪才返回。

此外，该函数的返回值为就绪文件符数量，最大不会超过maxevents。

**epoll的优化原理**：

尽管我们说epoll相比select/poll大大提升了性能，那它具体是怎么做的呢？第一点，也是最关键的一点，epoll不像select/poll需要轮询所有的文件描述符，而是要求内核将已就绪文件描述符放在一个就绪链表中，每次调用epoll_wait函数时就只需判断就绪链表是否为空即可。此外，epoll也不需要像select/poll每次调用都需要将待监视文件描述符从用户态复制到内核态，它只需要使用epoll_ctl函数注册一次即可。

**边缘触发和水平触发**：

epoll支持水平触发和边缘触发，而select/poll仅支持水平触发。

- 水平触发（LT，Level Trigger）：当文件描述符就绪时，会触发通知，如果用户程序没有一次性把数据读/写完，下次还会发出可读/可写信号进行通知。
- 边缘触发（ET，Edge Trigger）：仅当描述符从未就绪变为就绪时，通知一次，之后不会再通知。

二者相比，边缘触发效率更高。因为它减少了事件被重复触发的次数，函数不会返回大量用户程序可能不需要的文件描述符。

## 参考

- [Operating Systems: Three Easy Pieces: I/O Devices](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/36.pdf)
- [小林Coding - 图解系统 - 设备管理](https://www.xiaolincoding.com/os/7_device/device.html)
