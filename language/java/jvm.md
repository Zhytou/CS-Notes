# JVM

- [JVM](#jvm)
  - [JVM Runtime Data Areas](#jvm-runtime-data-areas)
    - [PC Register](#pc-register)
    - [JVM Stack](#jvm-stack)
    - [Native Method Stack](#native-method-stack)
    - [Heap](#heap)
    - [Method Area](#method-area)
    - [Runtime Constant Pool](#runtime-constant-pool)
    - [CG](#cg)
  - [Class Loader](#class-loader)
    - [Class File](#class-file)
    - [Class](#class)

## JVM Runtime Data Areas

因为Java可移植性，所以像C一样采用操作系统提供的运行时地址空间在JVM是不可取的。JVM提供了一套自己的运行时内存划分机制，它在执行Java程序时会把它管理的内存划分为若干个不同的数据区域，如图所示。

![JVM运行时数据区](https://pdai.tech/images/jvm/jvm/0082zybply1gc6fz21n8kj30u00wpn5v.jpg)

这些区域大体上可以按线程私有或共享分为两部分：

- 线程私有：程序计数器、虚拟机栈、本地方法区
- 线程共享：堆、方法区, 堆外内存（Java7的永久代或JDK8的元空间、代码缓存）

此外，JVM运行时数据区同样随着JDK版本变换而变化。如下图所示，JDK 1.6、JDK 1.7、JDK 1.8的内存划分都会有所不同。

![不同JDK的运行时数据区](https://s2.51cto.com/oss/202203/21/f7f83289195bc83d1be393d85e20885340b3df.png)

### PC Register

JVM中的程序计数寄存器(Program Counter Register)是对物理PC寄存器的一种抽象模拟，可以看作是当前线程所执行的字节码的行号指示器。

### JVM Stack

Java虚拟机栈，早期也叫Java栈。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)，对应着一次次 Java方法调用，是线程私有的，同时生命周期和线程一致。

**Stack Layout**：

虚拟机栈用于保存方法的局部变量、部分结果，并参与方法的调用和返回。它只涉及两个操作，每个方法执行时入栈，方法执行结束或抛出异常时出栈。因此，在任意一个时间点上，当前正在执行的方法都在栈顶栈帧，该栈帧也成为当前栈帧，其对应方法就当前方法。比如，一个嵌套调用函数的栈帧分布如下。

![multple stack frame layout](https://pdai.tech/images/jvm/jvm/0082zybply1gc8tjehg8bj318m0lbtbu.jpg)

可见，每个栈帧中都存储着：

- 局部变量表（Local Variables）
- 操作数栈（Operand Stack）(或称为表达式栈)
- 动态链接（Dynamic Linking）：指向运行时常量池的方法引用方法
- 返回地址（Return Address）：方法正常退出或异常退出的地址
- 附加信息

**Local Variable & Slot**：

**Dynamic Linking**：

### Native Method Stack

**JNI**：

Java本地接口(Java Native Interface)

**本地方法调用**：

类似虚拟机栈是用于管理Java方法的调用，而本地方法栈则是用于管理本地方法的调用。

### Heap

### Method Area

方法区是 JVM 中存储类和接口定义的共享数据区域。它在 JVM 启动时创建，并且仅在 JVM 退出时销毁。

具体来说，类加载器加载类的字节码并将其传递给 JVM。然后，JVM 创建类的内部表示，用于在运行时创建对象和调用方法。此内部表示收集有关类和接口的字段、方法和构造函数的信息。

此外，让我们指出，方法区是一个逻辑概念。因此，在具体的 JVM 实现中，它可能是堆的一部分。

再次，JVM 规范没有定义方法区的大小，也没有定义 JVM 处理内存块的方式。

### Runtime Constant Pool

运行时常量池是方法区内的一个区域，其中包含对类和接口名称、字段名称和方法名称的符号引用。

JVM 利用方法区中类或接口表示的创建来同时为该类创建运行时常量池。

创建运行时常量池时，如果 JVM 需要的内存多于方法区中可用的内存，则会引发 OutOfMemory 错误。

### CG

## Class Loader

### Class File

了解Java编译流程可知，Java程序并非类似C++的编译型语言，而是一种半编译语言。它会先将源程序编译成字节码文件，再使用解释器一行行执行。它往往包含：

魔数（Magic Number）：标识文件是一个Java字节码文件。
版本号（Version）：包含Java虚拟机版本和类文件版本。
常量池（Constant Pool）：存储各种常量，如字符串、类名、方法名、字段名等。
访问标志（Access Flags）：标识类或接口的访问权限和特性。
类索引（This Class）：指向常量池中的类或接口全名。
父类索引（Super Class）：如果类不是接口，指向父类的全名。
接口索引集合（Interfaces）：如果类是接口，列出实现的接口。
字段表集合（Fields）：类或接口的字段信息。
方法表集合（Methods）：类或接口的方法信息。
属性表集合（Attributes）：类、字段或方法的额外信息，如注解、源文件名等。

### Class

类加载过程
类加载器种类
内存管理
堆内存
方法区/元空间
栈内存
垃圾收集
GC 算法
GC 收集器
GC 调优
JIT 编译
热点代码
编译过程
调优工具
JVisualVM
JMC
JMH
JVM 监控
JMX
JVM 参数
JVM 日志
