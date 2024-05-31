# JVM

- [JVM](#jvm)
  - [JVM Runtime Data Areas](#jvm-runtime-data-areas)
    - [PC Register](#pc-register)
    - [JVM Stack](#jvm-stack)
    - [Native Method Stack](#native-method-stack)
    - [Heap](#heap)
    - [Method Area](#method-area)
    - [CG](#cg)
  - [Class Loader](#class-loader)
    - [Class File](#class-file)

## JVM Runtime Data Areas

JVM在执行Java程序时会把它管理的内存划分为若干个不同的数据区域，如图所示。

![JVM运行时数据区](https://pdai.tech/images/jvm/jvm/0082zybply1gc6fz21n8kj30u00wpn5v.jpg)

- 线程私有：程序计数器、虚拟机栈、本地方法区
- 线程共享：堆、方法区, 堆外内存（Java7的永久代或JDK8的元空间、代码缓存）

### PC Register

### JVM Stack

Java虚拟机栈，早期也叫Java栈。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)，对应着一次次 Java方法调用，是线程私有的，同时生命周期和线程一致。

虚拟机栈用于保存方法的局部变量、部分结果，并参与方法的调用和返回。它只涉及两个操作，每个方法执行时入栈，方法执行结束时出栈。比如，一个嵌套调用函数的栈帧分布如下。

![multple stack frame layout](https://pdai.tech/images/jvm/jvm/0082zybply1gc8tjehg8bj318m0lbtbu.jpg)

可见，每个栈帧中都存储着：

- 局部变量表（Local Variables）
- 操作数栈（Operand Stack）(或称为表达式栈)
- 动态链接（Dynamic Linking）：指向运行时常量池的方法引用方法
- 返回地址（Return Address）：方法正常退出或异常退出的地址
- 附加信息

### Native Method Stack

**JNI**：

Java本地接口(Java Native Interface)

**本地方法调用**：

类似虚拟机栈是用于管理Java方法的调用，而本地方法栈则是用于管理本地方法的调用。

### Heap

### Method Area

**常量池**：

### CG

## Class Loader

### Class File

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
