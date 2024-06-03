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

Java虚拟机栈，早期也叫Java栈。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)，对应着一次次Java方法调用。它是线程私有的，同时生命周期也和线程一致。

**Stack Layout**：

虚拟机栈用于保存方法的局部变量、部分结果，并参与方法的调用和返回。它只涉及两个操作，每个方法执行时入栈，方法执行结束或抛出异常时出栈。因此，在任意一个时间点上，当前正在执行的方法都在栈顶栈帧，该栈帧也成为当前栈帧，其对应方法就当前方法。比如，一个嵌套调用函数的栈帧分布如下。

![multple stack frame layout](https://pdai.tech/images/jvm/jvm/0082zybply1gc8tjehg8bj318m0lbtbu.jpg)

可见，栈帧存储了方法的局部变量表、操作数栈、动态连接和方法返回地址等信息。而其中的大部分信息都能在Class文件的方法表中找到对应的静态对照物。

**Local Variable Table**：

局部变量表(Local Variable Table)是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。当一个方法被调用时，它的参数和局部变量会被顺次复制到表中。一般来说，局部变量表所需要的容量大小是编译期确定下来的，并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的。此外，由于它是线程私有的，所以局部变量不存在线程安全问题。

局部变量表最基本的存储单元是变量槽(Slot)。JVM会为表中每个Slot都分配一个访问索引，通过这个索引即可快速访问变量和参数值。至于64位数据，则只需要使用第一个Slot索引即可。值得一提的是，栈帧中的局部变量表中的槽位是可以重用的，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而达到节省资源的目的。

**Operand Stack**:

操作数栈(Operand Stack)也常被称为操作栈，它是一个后入先出(Last In First Out，LIFO)栈。其主要作用就是保护计算过程的中间结果，同时作为计算过程中变量的临时存储空间，且调用方法的返回值也往往存储在调用者的操作栈中。而所谓JVM的解释引擎是基于栈的执行引擎就是指的就是操作数栈的出栈和入栈过程。比如，下图是局部变量表和操作数栈在计算减法时的变化。

![操作数栈减法](https://media.geeksforgeeks.org/wp-content/cdn-uploads/jvm.png)

此外，操作数栈的每一个元素都可以是包括long和double在内的任意Java数据类型。其中，32位数据类型所占的栈容量为1，64位数据类型所占的栈容量为2。

**Dynamic Linking**：

每个栈帧都包含一个指向方法区中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接(Dynamic Linking)。

Class文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以常量池里指向方法的符号引用作为参数。这些符号引用一部分会在类加载阶段或者第一次使用的时候就被转化为直接引用，这种转化被称为静态解析。另外一部分将在每一次运行期间都转化为直接引用，这部分就称为动态连接。

**Return Address**：

方法返回地址(Return Address)用于存放调用该方法的PC寄存器值。本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去。

### Native Method Stack

**JNI**：

Java本地接口(Java Native Interface)是一种连接本地代码和Java代码的机制，它允许二者相互调用。其中，本地代码一般是C/C++编写的。它将本地代码编译成平台相关的共享链接库(Shared Library)加载进JVM中，从而支持互相调用。

**本地方法调用**：

类似虚拟机栈是用于管理Java方法的调用，而本地方法栈则是用于管理本地方法的调用。此外，并非所有JVM都实现了本地方法栈，因为《Java虚拟机规范》中没有明确规定本地方法栈的使用语言、实现方法和数据结构等。如果不打算支持本地方法，也就无需实现本地方法栈。至于当前最流行的Hotspot JVM，它则是直接将本地方法栈和虚拟机栈合二为一。

### Heap

对于Java应用程序来说，Java堆是虚拟机所管理的内存中最大的一块。它被所有线程共享，且在虚拟机启动时创建。而其唯一目的就是存放对象实例，并由垃圾回收器进行统一管理。目前最主流的HotSpot虚拟机中，堆基于经典分代设计而被分为新生代、老年代和元空间（JDK1.8之前称为永久代）三部分。其中，元空间也就是HotSpot的方法区实现，尽管在《Java虚拟机规范》被称为非堆(Non-Heap)，但它实际上只是一个逻辑概念。

《Java虚拟机规范》指出堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可，像磁盘空间一样。实现时，既可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 -Xmx和-Xms控制），如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出OutOfMemoryError异常。

### Method Area

方法区(Method Area)与堆一样，是各个线程共享的内存区域。它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。虽然《Java虚拟机规范》中把方法区描述为堆的一个逻辑部分，但它却有一个别名非堆(Non-Heap)，目的是与Java堆区分开来。

方法区的大小和堆空间一样，可以选择固定大小也可选择可扩展，方法区的大小决定了系统可以放多少个类，如果系统类太多，导致方法区溢出，虚拟机同样会抛出内存溢出错误。

**方法区 vs 永久代 vs 元空间**：

方法区只是JVM规范中定义的一个概念，用于存储类信息、常量池、静态变量、JIT编译后的代码等数据，并没有规定如何去实现它，不同的厂商有不同的实现。而永久代(PermGen)是Hotspot虚拟机特有的概念，Java8的时候又被元空间(Metaspace)取代了，永久代和元空间都可以理解为方法区的落地实现。永久代物理是堆的一部分，和新生代、老年代地址是连续的，且受垃圾回收器管理；而元空间存在于堆外内存，不受垃圾回收器管理。

**JDK1.8 方法区变化**：

对于方法区，Java 8之后的变化包括：

- 移除了永久代，替换为元空间；
- 永久代中的class metadata转移到了堆外内存，不受JVM管理；
- 永久代中的interned Strings和class static variables转移到了Heap；
- 永久代参数(PermSize MaxPermSize)->元空间参数(MetaspaceSize MaxMetaspaceSize)。

### Runtime Constant Pool

运行时常量池(Runtime Constant Pool)是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表(Constant Pool Table)，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的是String.intern()方法。受方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

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
