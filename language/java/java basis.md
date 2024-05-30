# Java

- [Java](#java)
  - [Java Data Types \& Basic Data Structures](#java-data-types--basic-data-structures)
    - [Java Primitive Types](#java-primitive-types)
    - [Java Wrapper](#java-wrapper)
    - [Java String](#java-string)
    - [Java Array](#java-array)
  - [Java Collection \& Map](#java-collection--map)
    - [ArrayList](#arraylist)
    - [LinkedList](#linkedlist)
    - [PriorityQueue](#priorityqueue)
    - [HashSet/HashMap](#hashsethashmap)
    - [TreeSet/TreeMap](#treesettreemap)
    - [Iterator](#iterator)
  - [Java Lambda](#java-lambda)
  - [Java OOP](#java-oop)
    - [Constructor](#constructor)
    - [Modifier](#modifier)
    - [Abstraction](#abstraction)
    - [Polymorphism](#polymorphism)
    - [Package](#package)
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

## Java Data Types & Basic Data Structures

在Java中，一个变量的数据类型只可能是基本数据类型(Primitive Types)或引用数据类型(Reference Types)。两者异同可以参考[讨论](https://pages.cs.wisc.edu/~hasti/cs302/examples/primitiveVsRef.html)。

### Java Primitive Types

其中，基本数据类型包括：

- 布尔值(boolean)
- 整数值(byte、short、int、long)
- 浮点数值(float、double)
- 字符(char)

这些基本类型在内存中直接存储值，并且有固定的存储空间大小。值得注意的是，在使用数值类型时，需要添加一些额外的字符告知编译器数据类型，比如：

```java
int a = 100000;
long b = 101L;
float c = 0.2f;
double d = 0.00999d;
```

### Java Wrapper

在Java中，有不少场合都无法使用基本类型。比如：使用ArrayList存储int数据，就只能使用int的包装器Integer类。

```java
ArrayList<int> myNumbers = new ArrayList<int>(); // Invalid
ArrayList<Integer> myNumbers = new ArrayList<Integer>(); // Valid
```

这是因为ArrayList是一个泛型类，而Java泛型依靠将类型参数为类型上界(Object)实现，即模板参数必须要继承自Object才行，而int显然不满足条件，所以ArrayList只能使用Integer类存储int数据。关于这一点，更多更详细的解释可以参考stackoverflow的这两篇讨论[Why can't I have 'int' as the type of an ArrayList?](https://stackoverflow.com/questions/14349011/why-cant-i-have-int-as-the-type-of-an-arraylist)和[Can I use generics over reference types only?](https://stackoverflow.com/questions/3015716/can-i-use-generics-over-reference-types-only)。

**不可变性**：

因此，基本类型都有与之对应的包装器类型(Wrapper Class)，如Integer、Long等，用于基本类型与对象之间的转换。值得注意的是，包装器类型是不可变的，即已经创建就不可改变。比如：

```java
Integer a = new Integer(10);
Integer b = new Integer(11);
a = b;
// output 11
System.out.println(a);
```

个人感觉这一点有些类似C++中关于指针指向cosnt值的讨论，即指针的地址可以修改，但指向内容不可以修改。

**自动装箱和拆箱**：

装箱(Boxing)和拆箱(Unboxing)是Java中自动将基本数据类型与其对应的包装类进行转换的机制。装箱是指将基本数据类型转换为对应的包装类，而拆箱则是指将包装类转换为对应的基本数据类型。

装箱的原理是通过调用包装类的valueOf方法来实现。例如，对于Integer类型的装箱，可以通过调用Integer.valueOf(int)方法将int类型的值转换为对应的Integer对象。拆箱的原理是通过调用包装类的xxxValue方法（例如intValue、doubleValue等）来获取包装类中存储的基本数据类型的值。例如，对于Integer类型的拆箱，可以通过调用Integer.intValue()方法将Integer对象中的值转换为int类型的值。

为了提高性能和节省内存，当使用相同值创建包装器对象时，JVM会返回一个相同的对象引用，而不是创建一个新的对象。比如，JVM对范围在[-128, 127]之间的整型包装器就使用了缓存。这也是为什么下面代码的返回值分别为true和false。

```java
public class Main {
  public static void main(String[] args) {
    Integer i1 = 100;
    Integer i2 = 100;
    Integer i3 = 200;
    Integer i4 = 200;
    
    // true
    System.out.println(i1==i2);
    // false
    System.out.println(i3==i4);
  }
}
```

此外，JVM还对字符型包装器提供[0, 127]的缓存，对浮点型提供最小值和最大值两个缓存。换句话说，除了最大和最小两个值之外，每次创建浮点包装器时都会创建新的对象。关于自动装箱和拆箱相关的面试题，可以参考[博客](https://www.cnblogs.com/dolphin0520/p/3780005.html)。

### Java String

在Java中，String是一个类，并且是不可变的。换句话说，每当对String进行改变（如连接、替换等）时，都会创建一个新的String对象，而原来的String对象保持不变。这种设计使得String对象在多线程环境中是安全的。此外，每个String对象还内部包含一个字符数组，这个字符数组存储了实际的字符数据。

**Initialization**：

Java String有多种初始化方式，包括：

```java
// 直接赋值
String str1 = "Hello, World!";

// 使用构造函数
String str2 = new String("Hello, World!");

// 从字符数组创建
char[] chars = {'H', 'e', 'l', 'l', 'o'};
String str3 = new String(chars);

// 从字节数组创建
byte[] bytes = {72, 101, 108, 108, 111};
String str4 = new String(bytes);
```

其中，使用new关键字创建的String和直接赋值创建的String实际存储位置不同。前置存于堆，后者则存于JVM常量池，更详细的介绍可参考JVM内存管理笔记。

**Useful Functions**：

Java String提供了许多有用的函数，包括：检查字符串内容是否相等、连接字符串、查找和获取子字符串以及转换大小写等。

```java
String str = "Hello, World!";
String str1 = "Hello";
String str2 = "World";

/** 比较是否相等 */
// equals函数检查引用类型内容是否相等
boolean isEqual1 = str1.equals(str2);
// ==函数检查引用类型引用地址是否相等 
boolean isEqual2 = (str1 == str2);

/** 连接字符串 */
// 使用+操作符
String result1 = str1 + ", " + str2 + "!";
// 使用concat方法
String result2 = str1.concat(", ").concat(str2).concat("!");

/** 查找子字符串 */
int index = str.indexOf("World"); // 返回7

/** 获取子字符串 */
String substr = str.substring(7, 12); // "World"

/** 获取字符串长度 */
int length = str.length(); // 返回13

/** 转换为大写/小写 */
String upper = str.toUpperCase(); // "HELLO, WORLD!"
String lower = str.toLowerCase(); // "hello, world!"
```

### Java Array

在Java中，Array是一种用来存储固定大小相同数据类型元素的数据结构。在内存中，数组是一块连续的空间，可以通过索引来访问和操作数组中的元素。和String不同的是，Array即可以存储基本数据类型也可存储引用数据类型。

**Definition & Element Access**：

```java
// 一维数组
int[] arr1 = new int[5]; // 声明一个长度为5的int型数组
int[] arr2 = {1, 2, 3, 4, 5}; // 使用花括号直接初始化 
int[] arr3 = new int[] {1, 2, 3, 4, 5}; // 另一种初始化方式

// 二维数组 
int[][] arr2D = new int[3][4]; // 一个3行4列的二维数组
int[][] arr2D = {{1,2},{3,4},{5,6}}; // 直接初始化

// 访问数组首元素
System.out.println(arr2[0]); // 输出1

// 访问二维数组元素
System.out.println(arr2D[1][1]); // 输出4
```

**Useful Functions**：

此外，Array同样提供了许多有用的函数，包括：

```java
int[] arr1 = {1, 2, 3, 4};
int[] arr2 = {5, 6, 7, 8};
String[] strs = new String[5];

/** 获取数组长度 */
// 值得注意的是length是Array的属性，与String中的length()函数不同
int len = arr1.length;

/** 填充数组内容 */
// 直接修改strs内容，返回void
Arrays.fills(strs, "abc");

/** 检查数组内容是否相等*/
// 两个数组类型必须相同才能比较
Arrays.equals(arr1, arr2);

/** 排序数组 */
Arrays.sort(arr1);
```

## Java Collection & Map

在Java中，主要包括两种类型的容器：

- 集合（Collection）：用于存储单一元素；
- 图（Map）：用于存储键/值对映射。

其中，Collection接口又有3种子类型，分别是List、Set和Queue；而Map接口则有SortedMap和NavigableMap两个子接口。它们常用的实现类则包括：ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap等等。

![Java Collection](https://pdai.tech/images/java_collections_overview.png)

### ArrayList

ArrayList是一个可以动态修改的数组，与Array的最大区别就是它是没有固定大小的限制，可以添加或删除元素。此外，ArrayList只能存储引用数据类型，而Array则可以存储基本数据类型和引用数据类型。除了ArrayList之外，Collection接口还有一个名为Vector的实现。它同样是一个可以动态修改和没有固定大小限制的数组。只不过和ArrayList相比，它是线程安全的，且在扩容时往往按两倍原大小去申请内存，具体异同可参考stackoverflow的讨论[What are the differences between ArrayList and Vector?](https://stackoverflow.com/questions/2986296/what-are-the-differences-between-arraylist-and-vector)。

**Element Operation**:

ArrayList的核心功能就是元素操控，比如：添加、删除、修改、遍历等功能。

```java
ArrayList<String> arr = new ArrayList<String>();

/** 添加元素 */
arr.add("abc");
arr.add("xxx");

/** 访问元素 */
String s = arr.get(1);

/** 修改元素 */
arr.set(1, "yyy");

/** 删除元素 */
arr.remove(1);

/** 获取数组长度 */
int len = arr.size();
```

### LinkedList

LinkedList是一个双向链表的实现,它不是通过一个连续的内存空间来存储数据,而是使用一组对象来表示一个链,每个对象称为一个节点(Node),每个节点存储数据和两个指针(prev和next),分别指向前一个和后一个节点。

LinkedList同时实现了List接口和Deque接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列(Queue)，同时又可以看作一个栈(Stack)。这样看来，LinkedList简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用LinkedList，一方面是因为Java官方已经声明不建议使用Stack类，更遗憾的是，Java里根本没有一个叫做Queue的类(它是个接口名字)。关于栈或队列，现在的首选是ArrayDeque，它有着比LinkedList(当作栈或队列使用时)有着更好的性能。

```java
LinkedList<String> list = new LinkedList<>();

/** 添加元素 */
// 添加到链表尾部
list.add("d"); 
// 添加到链表头部 
list.addFirst("x");
// 添加到指定位置
list.add(2,"y");

/** 获取/修改元素 */
// 获取头部元素
String first = list.getFirst();
// 获取尾部元素 
String last = list.getLast();
// 获取指定位置元素
String e = list.get(2);
// 修改指定位置元素
list.set(2, "z");

/** 删除元素 */
// 删除头部元素
list.removeFirst();
// 删除尾部元素
list.removeLast(); 
// 删除指定元素
list.remove(2);
// 删除首次出现的"z"
list.remove("z");
```

### PriorityQueue

### HashSet/HashMap

HashMap是Java中Map接口的一种具体实现。它存储键值对映射，且允许null作为键或值。而HashSet则是Java中Collection子接口Set的一种具体实现，用于存储不重复的元素。事实上，HashSet的实现依赖于HashMap，即前者只是对后者的包装。因此，本节将主要聚焦于HashMap。

**Insertion**：

当HashMap被插入键值对时，它首先会根据键的hashCode计算该键值对应存放的索引位置，然后遍历索引位置处链表，查找是否存在与待插入键值对重复的键。如果没有找到重复键值对，就直接将键值对插入链表头部；如果找到了重复键，就替换旧值。

**Hash Collision**：

当发生哈希冲突时，HashMap会将冲突的键值对存储为同一个索引位置处的链表。当链表长度超过一定阈值(8)时，链表会转化为红黑树，有效避免了链表过长带来的哈希冲突风险。事实上，不同版本的JDK中HashMap实现是不同的：

- 在JDK1.7及之前版本，HashMap使用单链表解决哈希冲突。此版本下，链表过长性能会大幅下降。
- JDK1.8引入了更高效的哈希算法，并在桶中使用链表和红黑树组合，从而更好的解决了哈希冲突。但是此版本下，HashMap在并发环境下可能出现死循环问题。
- JDK1.9针对JDK1.8并发情况下死循环问题，对HashMap源码进行了改进。

**Differences Versus Other Map**：

和Map接口的其他实现相比，HashMap有以下特点：

- HashMap允许null键和null值，而TreeMap和Hashtable不允许null键。
- TreeMap是有序的，而HashMap和HashTable是无序的。
- HashTable是线程安全的，但HashMap不是。也正因如此，HashMap的性能优于HashTable。

**Useful Functions**：

```java
HashMap<String, String> map = new HashMap<String, String>();

/** 添加元素 */
// xxx不存在，创建xxx和yyy键值对
map.put("xxx", "yyy");
// xxx存在，覆盖原键值对
map.put("xxx", "zzz");

/** 获取值 */
String val = map.get("xxx");

/** 删除键值对 */
map.remove("xxx");

/** 获取大小 */
int s = map.size();

/** 遍历 */
for (String key : map.keySet()) {
  //...
}
for (String val : map.values()) {
  //...
}

HashSet<String> set = new HashSet<String>();
/** 添加元素 */
set.add("xxx");
// 添加两次xxx，set也只会存一个xxx
set.add("xxx");

/** 检查是否存在 */
set.contains("xxx");

/** 删除值 */
set.remove("xxx");

/** 获取大小 */
int s = set.size();

/** 遍历 */
for (String s : set) {
  //...
}
```

### TreeSet/TreeMap

### Iterator

## Java Lambda

## Java OOP

### Constructor

在Java中，构造器（Constructor）是一种特殊的方法，用于初始化新创建的对象。构造器与类同名，没有返回类型，包括默认构造器（无参数构造器）和带参数的构造器。构造器的主要作用是在创建对象时设置对象的初始状态，或者执行必要的初始化操作。

**默认构造器**：
当没有显式定义任何构造器时，Java会自动提供一个无参数的默认构造器。默认构造器不执行任何操作，只是调用父类的默认构造器（如果有的话）。

**构造器重载**：

Java允许开发者根据需要定义一个或多个带参数的构造器，以便在创建对象时传入初始值。这些构造器可以用来设置对象的成员变量，或者执行特定的初始化操作。只要这些构造器的参数数量、类型或顺序不同，这就是构造器重载。

**调用另一个构造器**：

在同一个类中，一个构造器可以调用另一个构造器，使用this()或this(args)。这允许共享构造器的代码，避免重复。

**super和this**：

super：super关键字用于引用父类的成员。它可以用于调用父类的构造器（super()或super(args)）、字段或方法。例如，super.someMethod()调用父类的someMethod()方法，super.someField引用父类的someField字段。
this：this关键字用于引用当前对象的成员。它可以用于调用当前类的构造器（this()或this(args)）、字段或方法。例如，this.someMethod()调用当前类的someMethod()方法，this.someField引用当前类的someField字段。

**继承链与构造器**：

当一个类继承自另一个类时，子类的构造器通常需要调用父类的构造器，以确保父类的状态也被正确初始化。这可以通过super()或super(args)来实现，其中super()调用无参数的父类构造器，super(args)调用带参数的父类构造器。

**静态工厂**：

类似C++，Java中也可以将构造器设为private从而实现单例模式。比如：

``` java
class Item {
  private static Item item = new Item();
  private Item() {}

  public static getItem() {
    return item;
  }
}
```

**字段初始化**：

在Java中，如果在构造器中没有为字段设置初值，那么它们就会自动地被赋予默认值。比如：数值为0、布尔值为false、对象引用为null。另外，在调用构造器之前，也可以直接在类定义中直接为每个字段设置初值，且这个初值不一定是常量值，也可以使用new或静态方法初始化。比如：

```java
class Employee {
  private static int nextId;
  private int id = assignId();
  private String name = "";

  private static assignId() {
    int r = nextId;
    nextId += 1；
    return r;
  }
}
```

**初始化块**：

除了上述提到的两种字段初始化方式：

- 在构造器中设置值；
- 在声明中赋值。

Java还提供了第三种机制，称为初始化块。在一个类的声明中，可以包含多个代码块。只要构造这个类的对象，这些块就会被执行。比如：

```java
class Employee{
  private static int nextId;
  private int id;
  private String name = "";

  {
    id = nextId;
    nextId++;
  }
}
```

此外，还有以static关键字标识的静态初始化块的代码块。它用于初始化类的静态成员，且无论创建多少个类的实例，它都只执行一次。值得一提的是，静态初始化块的执行顺序取决于它们在类中的位置，但通常在非静态初始化块和构造器之前执行。

**初始化顺序**：

- 首先，执行静态初始化块（如果有）。
- 然后，执行父类的构造器（如果有的话）。
- 接着，执行实例初始化块（如果有）。
- 最后，执行构造器。

### Modifier

前面代码示例中提到的关键字`public`是一个修饰符(Modifier)。在Java中，修饰符是一种用于修改类、方法、变量和其他实体的访问权限、行为或特性的关键字。一般来说，它们可以分为两类：

- 访问修饰符：public、private、protected等；
- 非访问修饰符：final、static、abstract等。

其中，每类修饰符又可按是否能用于修饰类分类。

**访问修饰符**：

default：当类没有添加访问修饰符时，默认使用default修饰，表示只能被该包内其他类或方法访问。
public：当其修饰类时，表示能被其他类访问到；当其修饰类内属性和方法方法时，表示可以通过该类对象访问。
private：只能用于修饰类内属性和方法，表示只能被该类中的其他方法访问。
protected：只能用于修饰类内方法和属性，表示只能被同一个包的方法和类或子类访问。

**非访问修饰符**：

abstract：当修饰类时，表示该类是抽象类，不能实例化对象；另外，还可用于修饰抽象类中无函数体的函数。
final：当修饰类时，表示不能被其他类继承；当修饰类内方法和属性时，表示不能被继承、重写或修改。
static：只能修饰类内属性和方法，表示可以直接使用类名访问，作用和C++中一致。
volatile：只能修饰属性和方法，表示访问时必须从主存中读取，作用和C++中一致。

### Abstraction

在Java中，抽象类(Abstract Class)和接口(Interface)都是用于实现数据抽象的概念。其中，抽象类就是使用关键字`abstract`修饰的类。它可以包含纯虚函数，即用关键字`abstract`修饰的无函数体函数。比如：

```java
// Abstract class
abstract class Animal {
  // Abstract method (does not have a body)
  public abstract void animalSound();
  // Regular method
  public void sleep() {
    System.out.println("Zzz");
  }
}

// Subclass (inherit from Animal)
class Pig extends Animal {
  public void animalSound() {
    // The body of animalSound() is provided here
    System.out.println("The pig says: wee wee");
  }
}
```

至于接口，则是一组无函数体函数的集合。它其实类似C++中纯虚类的概念，即只包含纯虚函数的类。相比抽象类，接口是层次更高的抽象类型。比如：

```java
// Interface
interface Animal {
  public void animalSound(); // interface method (does not have a body)
  public void sleep(); // interface method (does not have a body)
}

// Pig "implements" the Animal interface
class Pig implements Animal {
  public void animalSound() {
    // The body of animalSound() is provided here
    System.out.println("The pig says: wee wee");
  }
  public void sleep() {
    // The body of sleep() is provided here
    System.out.println("Zzz");
  }
}
```

除此之外，二者的异同还包括：

- 抽象类继承使用`extends`；接口继承使用`implements`。且Java中并不直接支持继承多个类，但支持同时继承多个接口。
- 接口中的属性默认是`final`的，但抽象类中属性可以使用`private`、`static`等修饰符修改。
- 接口中只能包含纯虚函数，而抽象类中可以包含非纯虚函数。

更详细的介绍参考geeksforgeeks上的讨论[Difference between Abstract Class and Interface in Java](https://www.geeksforgeeks.org/difference-between-abstract-class-and-interface-in-java/)。

### Polymorphism

在OOP中，多态(Polymorphism)指的是多个对象的同一方法表现出不同结果。类似C++，Java的多态也分为静态多态和动态多态，也被称为编译期多态(Compile-time Polymorphism)和运行时多态(Run-time Polymorphism)。其中，前者依赖方法重载(Overload)；后者依赖方法重写(Override)。

方法重载指的是同一个类中允许出现多个同名函数的情况，这些函数往往拥有不同数量或不同类型的参数。编译器会根据方法签名(方法名+参数列表)来确定实际调用函数。而方法重写则是指子类重新定义了从父类继承的一个方法。除了函数签名必须与父类一致外，重写允许将返回类型修改原返回类型的一个子类。当调用一个对象的属性或方法时，真正执行的方法仅在运行期才能确定，这就也就是动态多态。

关于二者更详细的异同，可以参考geeksforgeeks的讨论[Difference Between Method Overloading and Method Overriding in Java](https://www.geeksforgeeks.org/difference-between-method-overloading-and-method-overriding-in-java/)。至于这两个概念在C++和Java中差异，则可以参考Quora的讨论[What is the difference between method overloading in Java and C++?](https://www.quora.com/What-is-the-difference-between-method-overloading-in-Java-and-C)。

### Package

Java允许使用包(Package)将类组织在一个集合中。一般来说，开发者使用包来避免命名冲突，并且更方便维护。在实际项目中，我们往往使用因特网域名的逆序形式作为包名，比如：com.example.demo。此外，值得一提的是从Java编译器的角度来说，嵌套的包之间没有任何关系。比如，java.util和java.util.jar就毫无关系，每个包都是独立的集合。

**包的导入**：

一个类可以使用所属包中的所有类，以及其他包中的公共类(Public Class)。但是想要访问其他包中的公共类，就必须将其导入。第一种方式就是使用完全限定名，即包名跟着类名，比如：`java.time.LocalDate`。显然，这种方式很繁琐。Java提供了关键字`import`用于导入一个特定的类或整个包，比如：

```java
// 导入LocalDate
import java.time.LocalDate;
// 导入java.time包
import java.time.*
```

不过值得注意的是，使用星号只能导入一个包，而不能使用`import java.*`导入所有以java为前缀的包。

**在包中增加类**：

要想将类放入包中，就必须将包名放在源文件开头。比如：

```java
package com.example

public class xxx {

}
```

但是值得注意的是，每个单独的文件中都只能有一个public类，且必须与该文件名完全匹配。如果在某个文件中有一个以上的public类，编译器就会给出出错信息。

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
