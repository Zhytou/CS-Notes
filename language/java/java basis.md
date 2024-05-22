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
    - [Annotation Definition](#annotation-definition)
    - [Meta Annotation](#meta-annotation)
    - [Annotation Intrinsic](#annotation-intrinsic)
  - [Java IO](#java-io)
  - [Java Concurrency](#java-concurrency)

## Java Collection & Map

## Java OOP

## Java Reflection

## Java Generics

Java泛型是一种在编译时进行类型检查和类型安全的机制，它允许我们在创建类、接口和方法时使用参数化类型。通过使用泛型，我们可以在编译时指定要使用的具体类型，并在运行时保证类型的一致性。

泛型的主要好处是提高了代码的可读性、可维护性和安全性。它可以帮助我们避免类型转换错误，并提供了更好的代码重用性。以下是一个简单的Java泛型示例：

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}
```

与C++泛型相比，Java泛型在实现原理上存在一些不同之处：

- Java泛型是通过类型擦除来实现的，这意味着在编译时，泛型类型参数会被擦除为其上界（或Object类型）。这样做是为了保持与旧版本Java代码的兼容性，并允许泛型代码与非泛型代码进行互操作。
- C++泛型是通过模板实例化来实现的，编译器会根据每个使用的具体类型创建一个对应的模板实例。这意味着在C++中，泛型代码是以类型参数的实际类型进行编译的，因此可以实现更高的性能和更严格的类型检查。

尽管Java泛型在实现上与C++泛型有所不同，但它们的目标都是为了提供类型安全和代码重用性。Java泛型通过类型擦除来实现泛型代码与非泛型代码的兼容性，而C++泛型通过模板实例化实现更高的性能和更严格的类型检查。

## Java Dynamic Proxy

Java动态代理是Java语言提供的一种在运行时动态创建代理对象的机制。它可以在不修改源代码的情况下，对现有的对象进行增强或扩展。这种代理模式广泛应用于框架开发、AOP实现等领域。

### Proxy Design Pattern

代理模式是一种结构型设计模式，它为另一个对象提供了一个代理或占位符，以控制对该对象的访问。代理对象充当了实际对象和客户端之间的中介，从而实现了对实际对象的访问控制和增强。

### Static Proxy

与动态代理相对，静态代理是指在程序运行之前确定代理角色，并且明确代理类和目标类的关系。一般来说，在静态代理在，代理类和被代理类需要实现相同的接口，并在代理类中持有被代理对象的实例。比如：

```java
public class Proxy01 {
    public static void main(String[] args) {
        TargetObj targetObj = new TargetObj() ;
        ProxyObj proxyObj = new ProxyObj(targetObj) ;
        proxyObj.invoke();
    }
}
class TargetObj {
    public void execute (){
        System.out.println("目标类方法执行...");
    }
}
class ProxyObj {
    private TargetObj targetObj ;
    /**
     * 持有目标对象
     */
    public ProxyObj (TargetObj targetObj){
        this.targetObj = targetObj ;
    }
    /**
     * 目标对象方法调用
     */
    public void invoke (){
        before () ;
        targetObj.execute();
        after () ;
    }
    /**
     * 前后置处理
     */
    public void before (){
        System.out.println("代理对象前置处理...");
    }
    public void after (){
        System.out.println("代理对象后置处理...");
    }
}
```

这种模式有一个很显然的一个问题，即在实际的开发过程中，不可能为每个目标对象都定义一个代理类，同样也不能让一个代理对象去代理多个目标对象，这两种方式的维护成本都极高。

### JDK Proxy

JDK动态代理是基于接口实现的，它利用java.lang.reflect.Proxy类生成一个代理对象。代理对象与目标对象实现相同的接口，所有对代理对象的方法调用都会被重新路由到指定的InvocationHandler实例上。JDK动态代理的局限性是，只能为接口创建代理对象，不能为普通的类创建代理。

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

Java注解是一种元数据机制，它可以在源代码、编译时、运行时和部署时为程序元素（类、方法、字段等）添加额外的信息。注解可以用于提供配置信息、执行代码生成、进行编译时检查等。

### Annotation Definition

Java注解通过@interface关键字来定义。比如：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnnotation {
    String value() default "";
    int count() default 0;
}
```

在上述代码中定义了一个名为MyAnnotation的注解，通过@Retation指定它在运行时有效，通过@Target指定它用于方法上。此外，该注解具有两个属性：value和count，并且都有默认值。

**Annotation Attribute**：

从上面的例子可见，Java允许我们为注解定义属性。其一般语法如下`AttributeType AttributeName()`。并且，注解的属性可以通过default指定其默认值。

特别的，在Java注解中value()方法是一个特殊的方法，它被用作默认属性。如果注解只有一个属性，并且该属性名为value，我们可以省略属性名，直接指定值。这样，在使用注解时就可以直接写值，而无需指定属性名。

**Annotation Access**：

如果想要获取注解中的属性，我们可以通过反射的方式一步步取出注解，并且调用其中的属性方法。比如：

```java
import java.lang.reflect.Method;

public class AnnotationTest {
    @MyAnnotation(value="Hello World")
    public static void main(String[] args) {
        try {
            Class cls = AnnotationTest.class;
            Method method = cls.getMethod("main", String[].class);
            MyAnnotation anno = method.getAnnotation(MyAnnotation.class);
            System.out.println(anno.value());
        } catch (Exception ignore) {}
    }
}
```

### Meta Annotation

在MyAnnotation这个注解定义中还出现了两个注解@Target和@Retention，它们被称为元注解，也即注解的注解。Java中元注解共有以下这几个：

**@Target**：用于标识被修饰注解的作用。它的取值范围在ElementType这个枚举之中，包括：

```java
public enum ElementType {
    /** 类、接口、枚举定义 */
    TYPE,
    /** 字段，包括枚举值 */
    FIELD,
    /** 方法 */
    METHOD,
    /** 参数 */
    PARAMETER,
    /** 构造方法 */
    CONSTRUCTOR,
    /** 局部变量 */
    LOCAL_VARIABLE,
    /** 元注解 */
    ANNOTATION_TYPE,
    /** 包定义 */
    PACKAGE...
}
```

**@Retetion**：用于标识被修饰注解的生命周期。它的取值范围在RetetionPolicy枚举类之中，包括：

```java
public enum RetentionPolicy {
    /* 表示注解编译时可见，编译完后就被丢弃。这种注解一般用于在编译器做一些事情 */
    SOURCE,
    /* 表示在编译完后写入class文件，但在类加载后被丢弃。这种注解一般用于在类加载阶段做一些事情 */
    CLASS,
    /* 表示注解会一直起作用 */
    RUNTIME
}
```

**@Document**：用于标识被修饰的注解是否生成到API文档中。如果一个注解被@Documented注解修饰，那么当使用工具（如JavaDoc）生成API文档时，该注解的信息会包含在文档中。否则，注解的信息不会出现在文档中。

**@Inherited**：用于指示被修饰的注解是否可以被继承。如果一个注解被@Inherited注解修饰，并且被一个类继承，那么该注解也会被继承到子类中。注意，@Inherited只适用于类级别的注解，对方法、字段等注解无效。

### Annotation Intrinsic

**注解即接口**：

Java注解的本质是一个接口，它们继承自java.lang.annotation.Annotation接口。在编译时由编译器处理，并在运行时允许通过Java反射机制访问。它们不是语法糖，而是通过字节码文件中的特殊结构来存储和访问。换句话说，用于定义注解的关键字@interface实际和`public interface MyAnnotation extends java.lang.annotation.Annotation`是等价的。我们甚至可以使用javap反编译命令去验证一下。

![注解反编译](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vWmlwaHRyYWNrcy9GaWd1cmViZWQvcmF3L21hc3Rlci9pbWcvMS8yMDIwMDYwNjEwNDgyMC5wbmc?x-oss-process=image/format,png)

**内置注解**：

JDK提供了一些内置的注解，包括：

- @Override：检查重写方法
- @Deprecated：表示已过时，不推荐使用
- @SuppressWarnings：抑制警告
- @FunctionalInterface：标识函数式接口

**注解API**：

除了java.lang.annotation.Annotation之外，Java Annotation包还有两个重要的类就是之前提到的java.lang.annotation.ElementType和java.lang.annotation.RetentionPolicy。它们分别用于指定注解类型和作用域。

**注解与动态代理**：

- [Java Annotation的本质和实现原理](https://juejin.cn/post/6844904167517995022)
- [@interface的使用](https://blog.csdn.net/phelovhl/article/details/9797721)

## Java IO

## Java Concurrency
