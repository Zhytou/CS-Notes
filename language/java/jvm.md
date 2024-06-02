# JVM

- [JVM](#jvm)
  - [JVM Runtime Data Areas](#jvm-runtime-data-areas)
    - [PC Register](#pc-register)
    - [JVM Stack](#jvm-stack)
    - [Native Method Stack](#native-method-stack)
    - [Heap](#heap)
    - [Method Area](#method-area)
    - [Runtime Constant Pool](#runtime-constant-pool)
  - [Java Class File](#java-class-file)
    - [Class File Structure](#class-file-structure)
    - [ByteCode Instruction](#bytecode-instruction)
  - [Class Loading Mechanism](#class-loading-mechanism)
    - [Class Loading Process](#class-loading-process)
    - [Class Loader](#class-loader)
  - [CG](#cg)

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

方法区(Method Area)与堆一样，是各个线程共享的内存区域。它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

### Runtime Constant Pool

运行时常量池(Runtime Constant Pool)是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表(Constant Pool Table)，用于存放编译期生
成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

## Java Class File

了解Java编译流程可知，Java程序并非类似C++的编译型语言，而是一种半编译语言。它会先将源程序编译成字节码文件(ByteCode File)也即Class文件，再交由JVM执行。可见，Java实现平台无关性的基础就是JVM和字节码存储格式。JVM不与包括Java语言在内的任何程序语言绑定，它只与Class文件这种特定的二进制文件格式所关联，而Class文件中则包含了JVM指令集、符号表以及若干其他辅助信息。值得一提的是，如果一个Java源文件中有多个类，那么编译后会生成除内部类之外的每个类对应的Class文件。

### Class File Structure

**Magic Number**：

每个Class文件的头4个字节被称为魔数(Magic Number)，它的唯一作用就是标识文件是否是一个Java字节码文件。

**Version**：

紧接着魔数的4个字节存储的是Class文件的版本号(Version)。

**Constant Pool**：

版本号之后是常量池，它存储各种常量，如字符串、类名、方法名、字段名等。

访问标志（Access Flags）：标识类或接口的访问权限和特性。
类索引（This Class）：指向常量池中的类或接口全名。
父类索引（Super Class）：如果类不是接口，指向父类的全名。
接口索引集合（Interfaces）：如果类是接口，列出实现的接口。
字段表集合（Fields）：类或接口的字段信息。
方法表集合（Methods）：类或接口的方法信息。
属性表集合（Attributes）：类、字段或方法的额外信息，如注解、源文件名等。

### ByteCode Instruction

## Class Loading Mechanism

JVM把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这个过程被称作虚拟机的类加载机制。

与那些在编译时需要进行连接的语言不同，在Java语言里面，类型的加载、连接和初始化过程都是在程序运行期间完成的，这种策略让Java语言进行提前编译会面临额外的困难，也会让类加载时稍微增加一些性能开销，但是却为Java应用提供了极高的扩展性和灵活性，Java天生可以动态扩展的语言特性就是依赖运行期动态加载和动态连接这个特点实现的。例如，编写一个面向接口的应用程序，可以等到运行时再指定其实际的实现类，用户可以通过Java预置的或自定义类加载器，让某个本地的应用程序在运行时从网络或其他地方上加载一个二进制流作为其程序代码的一部分。

**类生命周期**：

一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)七个阶段，其中验证、准备、解析三个部分统称为连接（Linking）。上述7个阶段发生顺序如图所示。

![类生命周期](https://pdai.tech/images/jvm/java_jvm_classload_2.png)

**加载和初始化的时机**：

关于在什么情况下需要开始类加载过程的第一个阶段“加载”，《Java虚拟机规范》中并没有进行强制约束，这点可以交给虚拟机的具体实现来自由把握。但是对于初始化阶段，《Java虚拟机规范》则是严格规定了有且只有六种情况必须立即对类进行初始化：
1）遇到new、getstatic、putstatic或invokestatic这四条字节码指令时，如果类型没有进行过初始
化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有：
·使用new关键字实例化对象的时候。
·读取或设置一个类型的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）
的时候。
·调用一个类型的静态方法的时候。

### Class Loading Process

在加载阶段，Java虚拟机需要完成以下三件事情：
1）通过一个类的全限定名来获取定义此类的二进制字节流。
2）将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3）在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

### Class Loader

## CG
