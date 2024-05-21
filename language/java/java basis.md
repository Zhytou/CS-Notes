# Java

- [Java](#java)
  - [Java Collection \& Map](#java-collection--map)
  - [Java OOP](#java-oop)
  - [Java Reflection](#java-reflection)
  - [Java Generics](#java-generics)
  - [Java Dynamic Proxy](#java-dynamic-proxy)
    - [Proxy Design Pattern](#proxy-design-pattern)
    - [Static Proxy](#static-proxy)
    - [JDK Proxy](#jdk-proxy)
    - [CGLIB Proxy](#cglib-proxy)
  - [Java Compilation](#java-compilation)
    - [Java Compiler Frontend](#java-compiler-frontend)
    - [JIT](#jit)
    - [AOT](#aot)
  - [Java Annotation](#java-annotation)
  - [Java IO](#java-io)
  - [Java Concurrency](#java-concurrency)

## Java Collection & Map

## Java OOP

## Java Reflection

## Java Generics

## Java Dynamic Proxy

Java动态代理是Java语言提供的一种在运行时动态创建代理对象的机制。它可以在不修改源代码的情况下，对现有的对象进行增强或扩展。这种代理模式广泛应用于框架开发、AOP实现等领域。

### Proxy Design Pattern

代理模式

### Static Proxy

### JDK Proxy

### CGLIB Proxy

## Java Compilation

常见的编译型语言如C++，通常会把代码直接编译成CPU所能理解的机器码来运行。而Java被归类为一种半编译型语言，这是因为它为了实现“一次编译，处处运行”的特性，把编译的过程分成两部分。首先它会先由javac编译成通用的中间形式——字节码，然后再由解释器逐条将字节码解释为机器码来执行。所以在性能上，Java通常不如C++这类编译型语言，但优于Python这类解释型语言。

### Java Compiler Frontend

Java的执行过程整体可以分为两个部分，第一步由javac将源码编译成字节码，在这个过程中会进行词法分析、语法分析、语义分析，编译原理中这部分的编译称为前端编译。接下来无需编译直接逐条将字节码解释执行，在解释执行的过程中，虚拟机同时对程序运行的信息进行收集，在这些信息的基础上，编译器会逐渐发挥作用，它会进行后端编译——把字节码编译成机器码，但不是所有的代码都会被编译，只有被JVM认定为的热点代码，才可能被编译。

怎么样才会被认为是热点代码呢？JVM中会设置一个阈值，当方法或者代码块的在一定时间内的调用次数超过这个阈值时就会被编译，存入codeCache中。当下次执行时，再遇到这段代码，就会从codeCache中读取机器码，直接执行，以此来提升程序运行的性能。

### JIT

为了优化Java的性能，JVM在解释器之外引入了即时（Just In Time, JIT）编译器：当程序运行时，解释器首先发挥作用，代码可以直接执行。随着时间推移，即时编译器逐渐发挥作用，把越来越多的代码编译优化成本地代码，来获取更高的执行效率。解释器这时可以作为编译运行的降级手段，在一些不可靠的编译优化出现问题时，再切换回解释执行，保证程序可以正常运行。

即时编译器极大地提高了Java程序的运行速度，而且跟静态编译相比，即时编译器可以选择性地编译热点代码，省去了很多编译时间，也节省很多的空间。目前，即时编译器已经非常成熟了，在性能层面甚至可以和编译型语言相比。不过在这个领域，大家依然在不断探索如何结合不同的编译方式，使用更加智能的手段来提升程序的运行速度。

### AOT

除了JIT之外，Java还支持预期编译(Ahead-Of-Time, AOT)技术。AOT编译器可以在程序运行之前，将Java字节码直接编译为本地机器码，从而避免了JIT编译的开销，提高了启动速度。不过，AOT编译器优化能力通常不如JIT编译器，因此对于启动后长期运行的服务端程序，JIT编译更加合适。

## Java Annotation

- [Java Annotation的本质和实现原理](https://juejin.cn/post/6844904167517995022)
- [@interface的使用](https://blog.csdn.net/phelovhl/article/details/9797721)

## Java IO

## Java Concurrency
