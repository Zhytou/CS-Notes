# 内存模型

- [内存模型](#内存模型)
  - [进程空间](#进程空间)
    - [用户空间](#用户空间)
    - [内核空间](#内核空间)
  - [动态内存申请](#动态内存申请)
    - [new与malloc 的区别](#new与malloc-的区别)
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

### new与malloc 的区别

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
