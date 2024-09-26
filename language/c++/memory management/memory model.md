# 内存模型

- [内存模型](#内存模型)
  - [进程空间](#进程空间)
    - [用户空间](#用户空间)
    - [内核空间](#内核空间)
  - [动态内存申请](#动态内存申请)
    - [mmap](#mmap)
    - [malloc实现](#malloc实现)
    - [new与malloc的区别](#new与malloc的区别)
    - [new与数组](#new与数组)
    - [std::allocator](#stdallocator)
  - [参考](#参考)

了解C/C++程序中变量和进程的内存模型对于理解程序的行为和性能优化非常重要。下面会分别介绍进程空间、对象声明周期和申请动态内存三部分知识。

## 进程空间

**提出背景**：

为了多个程序能同时驻留在内存且互不干扰地运行，操作系统使用虚拟内存技术提供了一种地址空间的机制（Address Space）来保证隔离性。通过地址空间，操作系统向上层应用屏蔽了存储细节，完成了对物理内存的抽象。换句话说，进程空间就是程序能够看到的内存。

因此，掌握进程空间对于写出高性能程序十分重要。

**总线宽度与地址空间**：

在32位Linux系统上，一个进程的地址空间如图1所示。一般来说，32位系统上的进程空间为4GB（2^32Bytes），而64位系统上进程空间理论上能达到16EB（2^64Byte）。但由于操作系统和硬件的限制，一般达不到那个数目。比如，在64Linux系统上，仅使用低47位寻址，高17位做扩展（只能是全0或全1），即地址空间大小为2^47Bytes。且在中间还有大段未使用地址，具体可以查看[这篇CSDN博客](https://blog.csdn.net/hbuxiaofei/article/details/109517919)。

![图1 进程地址空间](https://gabrieletolomei.files.wordpress.com/2013/10/program_in_memory2.png)

### 用户空间

一个进程空间包含了程序的所有运行状态，并且总体上可以分成两大部分，即用户空间和内核空间。其中，前者也就是实现进程隔离性的核心。而后者虽然是共享的（映射了同一段物理内存），但仅仅只有当进程需要执行系统调用，并切换到内核态之后才会真正访问。

对于用户空间部分，它一般由以下几个部分组成：

- Text段：存储程序的可执行指令代码；
- Data段：存储已初始化的全局变量和静态变量；
- BSS段(Block Starting Symbol Segement)：存储未初始化的全局变量和静态变量（计算机自动用0给这些变量初始化）；
- Stack段：存储函数调用时的返回地址、参数、局部变量等；
- Heap段：用于动态分配的内存区域。

初此之外，还有一个Rodata段未在上图中标注出来，它用于存放虚表和常量字符串和虚表。

### 内核空间

而对于内核空间，它则一般包含：

- 内核代码和数据
- 系统调用表
- 设备驱动
- 中断处理程序
- 进程描述符
- 缓冲区
- 设备寄存器虚拟映射表

**内核的物理地址**：

尽管内核空间在进程空间中处于高位地址，它的物理地址却总是0x00开始。

**内核空间的作用**：

内核空间是操作系统的特权模式，在这个模式下，操作系统具有对系统资源的完全控制权，包括：CPU、内存、设备等。内核空间中运行的代码可以执行特权指令，访问受保护的资源，并执行敏感的操作。

与内核空间相对应的是用户空间，用户空间是供用户进程运行的区域。用户进程在用户空间中运行，只能访问受操作系统授权的资源，并且不能直接访问内核空间。

**使用内核空间的前提**：

进程在操作系统中是处于一种受限制直接执行（Limited Direct Execution）的模式之中的。其中，直接执行十分好理解，即程序直接作用于CPU。而受限制则是指进程默认处于用户模式（User Mode），无法访问一些硬件资源。只有当其使用陷入（Trap）内核指令切换到内核模式（Kernel Mode），才能访问机器的所有硬件资源。此外，操作系统还提供了一种陷阱返回（Return From Trap）指令来支持内核模式到用户模式的切换。

尽管操作系统通过用户和内核两种模式，使得进程在需要一些特殊硬件资源时，必须将CPU的控制权返还给操作系统，但这仅仅只能覆盖部分情况，当进程不需要访问硬件资源时，操作系统又该怎么做呢？答案是，利用时钟中断（Timer Interrupt）。当产生一个时钟中断时，当前正在运行的进程停止，操作系统中预先配置的中断处理程序（Interrupt Handler）会运行。此时，操作系统重新获得CPU的控制权。这也就是操作系统进程切换的底层机制。

## 动态内存申请

### mmap

> [认真分析mmap：是什么 为什么 怎么用](https://www.cnblogs.com/huxiao-tee/p/4660352.html)

**函数原型**：

mmap是一种系统调用，用于在进程的虚拟地址空间中创建一个内存映射区域，将文件或者设备映射到进程的地址空间中。当mmap成功执行时，返回被映射区的指针。失败时，返回-1，同时将errno设置为对应错误。其函数原型如下：

```c++
#include <sys/mman.h>
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
```

其中，mmap参数意义如下：

- start表示映射区起始地址。
- length表示映射区长度。
- prot表示内存保护标志，不能与文件的打开模式冲突，可以是下列某个值或通过or运算组合：
  - PROT_EXEC：页内容可以被执行；
  - PROT_READ：页内容可以被读取；
  - PROT_WRITE：页可以被写入；
  - PROT_NONE：页不可访问；
- flags表示映射对象的类型，映射选项和映射页是否可以共享。常见的映射类型包括：
  - MAP_ANONYMOUS：匿名映射，映射区不与任何文件关联。
  - MAP_LOCKED：锁定映射区的页面，从而防止页面被交换出内存。
  - MAP_SHARED：与其它所有映射这个对象的进程共享映射空间。对共享区的写入，相当于输出到文件。直到msync()或者munmap()被调用，文件实际上不会被更新。
- fd表示的文件描述符，可以是fopen等函数返回值。当flags为MAP_ANONYMOUS时，fd应当为-1。
- offset表示被映射对象内容的起点。

**和常规文件操作的区别**：

正如上一节所提到的那样，mmap可以实现文件映射，从而直接在用户空间对文件进行写入或读取。它和普通文件操作相比，执行速度更快且消耗更小。

具体来说，普通的文件操作，如fread/fwrite，执行过程包括：

- 进程发起读文件请求。
- 内核通过查找进程文件符表，定位到内核已打开文件集上的文件信息，从而找到此文件的inode。
- inode在address_space上查找要请求的文件页是否已经缓存在页缓存中。如果存在，则直接返回这片文件页的内容。
- 如果不存在，则通过inode定位到文件磁盘地址，将数据从磁盘复制到页缓存。之后再次发起读页面过程，进而将页缓存中的数据发给用户进程。

由上可知，由于缓存处于内核空间之中，使用fread/fwrite进行文件读取/写入的用户进程必须通过两次拷贝获取实际数据。相比之下，mmap通过建立物理地址（磁盘）到虚拟地址（内存）的映射，在实际读取文件时直接触发缺页中断，仅需一次拷贝即可将数据传入用户空间中。

**创建动态内存**：

除了读写文件之外，通过将flags设置为MAP_ANONYMOUS（匿名映射）mmap还能创建动态内存。它也是malloc的底层实现之一。

### malloc实现

> [malloc-mmap-brk](https://www.cnblogs.com/beilou310/p/17037066.html)

### new与malloc的区别

| Feature               | new/delete             | malloc/free                |
| --------------------- | ---------------------- | -------------------------- |
| Memory allocated from | 'Free Store'           | 'Heap'                     |
| Return                | Fully typed pointer    | void *                     |
| On failure            | Throws                 | Returns NULL               |
| Required size         | Calculated by compiler | Must be specified in bytes |

> Technically, memory allocated by `new` comes from the 'Free Store' while memory allocated by `malloc` comes from the 'Heap'. Whether these two areas are the same is an implementation detail, which is another reason that `malloc` and `new` cannot be mixed.

### new与数组

为了让new分配一个对象数组，我们需要在类型名后跟一对方括号，并在其中指明需要分配的数目。例如：

```c++
// 在堆上申请了10个int
int* p = new int[10];
```

值得注意的是，分配的数目为0时，编译也可以通过。此时得到的指针被称为尾后指针，类似标准库容器中end函数得到的迭代器。

**释放动态数组**：

为了释放动态数组，我们需要使用一种特殊形式的delete，即：在指针前加一个空的方括号。例如：

```c++
// 销毁p指向的第一个元素
delete p;
// 销毁p指向的动态数组
delete [] p;
```

### std::allocator

标准库allocator类定义在头文件memory中，它帮助我们将内存分配和对象构造分离开来。（因为调用new时，必须执行类的构造函数）std::allocator分配的内存是原始的、未构造的。

## 参考

**new 与 malloc 的区别**:

- [stackoverflow](https://stackoverflow.com/questions/240212/what-is-the-difference-between-new-delete-and-malloc-free)
