# 指令集架构

- [指令集架构](#指令集架构)
  - [分类](#分类)
    - [复杂指令集架构](#复杂指令集架构)
    - [精简指令集架构](#精简指令集架构)
  - [参考](#参考)

指令集架构（Instruction Set Architecture）是计算机体系结构中与程序设计有关的部分。它包含了基本数据类型、指令集、寄存器模式、寻址模式、中断集、异常处理以及外部I/O等。

简单来说，指令集架构定义了一台可运算的机器的机器指令。

## 分类

### 复杂指令集架构

Complex Instruction Set Computer CISC：

复杂指令集架构是一种微处理器指令集架构。每个指令可以指向若干低端操作，即若干操作集于单一指令中。复杂指令集的特点是指令数目多而复杂，每条指令字长并不相等，电脑必须加以判读，并为此付出了性能的代价。基于CISC架构的处理器有x86、x64家族等。

**x86 vs x64**：

x86 和 x64都是CISC.

> x86 refers to a 32-bit CPU and operating system while x64 refers to a 64-bit CPU and operating system.

### 精简指令集架构

Reduced Instruction Set Computer RISC**：

这种指令集的特点是指令数目少，每条指令都采用标准字长、执行时间短、中央处理器的实现细节对于机器级程序是可见的等等。RISC设计的根本原则——针对流水线化的处理器优化——没有改变。

**MIPS**：

MIPS架构也是一种RISC，是一般课程的重要教学材料。

**ARM**：

ARM架构是一种RISC，过去称作Advanced RISC Machine。

ARM架构处理器占市面上所有32位嵌入式RISC处理器90%的比例。

## 参考

**ISA**：

- [wiki - Instruction Set Architecture](https://en.wikipedia.org/wiki/Instruction_set_architecture)
- [ISA vs Machine Language](https://www.cise.ufl.edu/~mssz/CompOrg/CDA-lang.html)

**x86 vs x64**：

- [x86 vs x64](https://www.seeedstudio.com/blog/2020/02/24/what-is-x86-architecture-and-its-difference-between-x64/)
