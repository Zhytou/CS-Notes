# 元编程

- [元编程](#元编程)
  - [模板编程](#模板编程)
    - [宏与泛型](#宏与泛型)
    - [模板函数和类](#模板函数和类)
    - [可变模板参数](#可变模板参数)
    - [参数类型推导和完美转发](#参数类型推导和完美转发)
    - [特化和SFINAE](#特化和sfinae)
  - [深入constexpr](#深入constexpr)
    - [constexpr常量](#constexpr常量)
    - [constexpr模板常量](#constexpr模板常量)
    - [if constexpr](#if-constexpr)
  - [编译期运算](#编译期运算)
    - [数值计算](#数值计算)
    - [类型计算](#类型计算)
    - [编译时多态](#编译时多态)
    - [静态反射](#静态反射)
  - [参考](#参考)

**元编程定义**：

元编程指的是编写那些能够操作或生成代码的代码。一些常见的元编程表现形式包括：

- 反射：程序能够在运行时获取自身以及其他程序信息；
- 宏：程序经过预处理操作后会自动生成或修改代码；
- 面向切面编程：通过插件机制将代码分割成不同的关注点，在编译或运行期间动态组合代码模块。

**C++元编程**：

在C++中，元编程的流派可以分为：

- 模板元编程
- Constexpr all the things!
- 两者结合

其中，模板编程指的使用C++模板来在编译期生成对应类型的代码从而复用一份代码，而constexpr技术则更偏向编译期计算。

**泛型编程**：

泛型编程和元编程从概念上并无直接联系，它指的是编写可复用逻辑的代码，这些代码可以适用于多种类型的数据。但比较巧合的是，C++模板编程就既可以实现泛型编程，又可以实现元编程，所以让人容易混乱。

## 模板编程

模板起初是为了支持泛型替代而设计的语法。它的原理和宏类似，在编译期根据实际调用时传入的类型参数替换占位符，生成专属该类型的代码。但和宏相比，它具有类型安全、编译期检查错误以及零成本抽象的特点。

### 宏与泛型

在模板语法还未出现之时，依靠宏来实现泛型容器的方法如下：

```c++
// collection.h
#define DECLARE_COLLECTION(TYPE) \
struct Collection_##TYPE { \
  TYPE *arr; \
  unsigned int size, cap; \
};\
Collection_##TYPE make_Collection_##TYPE(unsigned int sz) { \
  Collection_##TYPE coll; \
  coll.arr = new TYPE[2*sz]; \
  coll.size = sz; \
  coll.cap = 2*sz; \
  return coll; \
}

// main.cpp
#include "collection.h"

DECLARE_COLLECTION(int);
DECLARE_COLLECTION(double);

int main() {
  Collection_int ci = make_Collection_int(5); 
  Collection_double cd = make_Collection_double(5);
}
```

利用宏，实现了对collection.h中代码的复用。

### 模板函数和类

随着模板语法的出现，我们可以使用它重写上述collection类，比如：

```c++
template<typename T>
struct Collection {
  T* arr;
  size_t size, cap;

  Collection(unsigned int sz) : size(sz), cap(2*cap) {
    arr = new T[cap];
  }
};
```

**关键字**：

其中，template是模板关键字，而typename则是模板参数关键字。前者用于声明一个模板函数或类；后者则用于声明可能出现的参数类型，并且可以用class替代。

除此之外，template中还可以直接传入常量参数，比如std::array要求传入一个非负整数用于指定固定数组长度。

```c++
template<class T, std::size_t N>
struct array;

// 定义了一个存储int、长度为3的数组
std::array<int, 3> arr;
```

**参数替换——零成本实现**：

> C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

C++模板遵循零成本实现原则。因为模板只会根据实际调用时传入的类型参数替换占位符，生成专属该类型的代码。如果没有该类型的调用，则不会发生。

### 可变模板参数

一个可变参数模板(variadic template)就是一个接受可变数目参数的模板函数或者模板类。可变数目的参数被称为参数包(parameter packet)。我们用一个省略号来指出一个可变数目参数。例如：

``` c++
template<typename T, typename... Args>
void func(const T& t, const Args& ... rest);
```

当我们想知道参数包中有多少参数时，我们可以使用sizeof...()运算符计算。

**递归可变参数函数**：

一般来说，可变参数函数通常是递归的。例如：

``` c++
// 非可变参数版本用于终止递归
template<typename T>
ostream& print(ostream& os, const T& t) {
  os << t;
}

template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args... rest) {
  os << t;
  print(os, rest);
}
```

**包扩展**：

对于一个参数包，除了能获取其大小外，我们能对它做的唯一事情就是扩展(expand)它。扩展一个包就是将它分解为构成元素，对每个元素应用模式，获得扩展后的列表。我们通过在模式右边放一个省略号来触发扩展操作。

例如，下面的print函数包含了两个扩展：

``` c++
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args... rest) { // 扩展
  os << t;
  print(os, rest);// 扩展
}
```

### 参数类型推导和完美转发

前面提到C++模板编程的基础是替换，即编译器会根据实际的模板参数来生成对应的代码。在这个过程中，参数类型的推导就决定了可能会生成什么类型的代码。下面来看几个参数推导的例子。

**左值引用参数推断**:

我们首先来看模板函数中参数为左值引用(T&)的情况。

``` c++
template <typename T> void f(T&);//实参必须是一个左值
f(i);  //i是一个int，模板参数T是int
f(5);  //错误，实参必须是左值
```

从上可见，当模板函数参数为左值引用(T&)时，它要求传入参数必须是一个左值，否则就会编译报错。

**右值引用参数推断**：

当模板参数为右值引用(T&&)时，它可以接收任何参数，所以也得名万能引用。比如：

```c++
int i;
int& ii = i;
template <typename T> void f(T&&);
f(42); // 实参是一个int类型的右值，模板参数T是int
f(i); // 正确
f(ii); // 正确
```

看上述例子，i作为一个int对象而ii作为一个int引用，理论上f(i)和f(ii)这样的调用应该不合法。毕竟，我们一般不会将一个右值引用绑定到一个左值或左值引用上去。但C++定义了两种特殊的规则允许这种绑定。

- 将一个左值传递给右值引用参数(T&&)时，编译器推断模板类型参数为实参的左值引用类型(T&)。换句话说，调用f(i)时，编译器推断T的类型为int&。
- 将一个左值引用传递给右值引用参数(T&&)时，编译器会进行引用折叠，得到模板类型参数为实参的左值引用类型(T&)。换句话说，调用f(ii)时，编译器推断T的类型也为int&。

**引用折叠**：

事实上，上述参数推断只用到了部分引用折叠规则。完整的引用折叠规则包括：

- X& &、X& && 和 X&& & 都会折叠为 X&。
- X&& && 会折叠为 X&&。

**什么是转发**：

在C++中，转发指的就是函数之间的参数传递。有些时候，参数在函数间传递时属性可能会发生改变。比如：

```c++
void innerFunc1(int& x) {
  x += 1;
}

void func1(int x) {
  innerFunc1(x);
}

int main() {
  func1(0);
}
```

上述例子中，innerFunc1对参数的修改无法传递到main函数中，因为innerFunc1得到的只是func1生成的一个副本。又比如：

```c++
void innerFunc2(int&& x) {
  // ...
}

void func2(int&& x) {
  innerFunc2(x);
}

int main() {
  func2(2);
}
```

这个例子的代码甚至不能编译通过，因为func2中的x参数是一个右值引用，即左值（对，没错，右值引用是一个左值，因为它可以求地址），而innerFunc2要求传入一个右值。

此时，我们即需要一种方法能按照参数原来的类型转发到另一函数，即完美转发。

**完美转发和std::forward**：

C++标准库定义了std::forward函数来保证实现完美转发。具体来说，std::forward的功能是：当参数不是一个左值引用时，函数返回一个右值引用；当参数是一个左值引用时，函数什么都不修改直接返回。

而模板编程中参数推断的规则，尤其是万能引用(T&&)的推断则是std::forward得以实现的基础。

### 特化和SFINAE

模板特化(Specialization)是指针对特定类型或特定条件，对模板进行定制化的实现。通过模板特化，我们可以为特定类型或条件提供特定的实现，从而覆盖模板的通用实现。

**全特化**:

全特化是指针对模板参数某种完整且明确类型的定义。例如：

``` c++
// 模板
template<typename T>
class XXX{
  ...
};

// 全特化模板
template<>
class XXX<int>{
  ...
};
```

**偏特化**:

偏特化是指针对模板参数进行更进一步的条件限制所设计出来的一个特化版本。例如：

``` c++
// 模板
template<typename T>
class XXX{
  ...
};

// 偏特化模板
template<typename T>
class XXX<T*>{
  ...
};
```

**SFINAE**：

替换失败并非错误(Substitution Failuer Is Not An Error, SFINAE)是C++的一种编译机制。在SFINAE机制下，编译器不会将模板替换失败视为错误，而是会尝试继续查找其他可行的实例。这种特性就允许C++程序根据条件启用或禁用萃取，进而是模板生成最匹配的代码。比如：

```c++
template<typename T>
enable_if_t<is_integral_v<T>> print(T val) {
  cout << "int:" << val << endl;
}

template<typename T>
enable_if_t<is_floating_point_v<T>> print(T val) {
  cout << "float:" << val << endl;
}

int main() {
  // int:1
  print(1);
  // float:1
  print(1.0f);
}
```

SFINAE和模板特化相比，它是指在模板实例化时根据条件选择是否生成特定版本的模板，而特化通常用于对特定类型或条件进行定制化实现。

## 深入constexpr

**constexpr发展**：

- C++11 引入constexpr简单函数
  - 只允许一条return语句
  - 递归解决问题！简单的数学函数、字符串hash函数
- C++14 放开constexpr约束, 模板变量
  - 泛化constexpr
  - 一些库出现
- C++17 if constexpr、constexpr lambda、折叠表达式
  - 表达力提升
- C++20 constexpr容器、constexpr new、constexpr析构函数、constexpr虚函数、consteval/constinit、lambda模板参数
- constexpr STL algorithms

**constexpr vs 模板**：

- constexpr作为新势力，生态待完善，参考资料少；而模板则体系成熟，拥有大量库；
- constexpr可读性强，维护性强；而模板可读性差，维护性差；
- constexpr编译速度快，跨平台支持强；而模板编译速度慢，跨平台支持弱。

### constexpr常量

**constexpr vs const**：

和const要求结果不可修改不同，constexpr变量必须能在编译期求值得到结果，而constexpr函数则要求函数体内所有计算必须能在编译期完成求值。

换句话说，const只保证引用期不变，但不能保证编译期求值，而constexpr则可以在编译期计算和优化，获得更高效能。

**constexpr lambda**：

C++17起，lambda默认为constexpr，比如：

```c++
// constexpr int fibonacci(int n);
auto fibonacci = [](int n) {
    int a = 0, b = 1;
    for (int c = 0; c < n; ++ c) {
        int t = a + b;
        a = b;
        b = t;
    }
    return a;
};

constexpr auto v = fibonacci(10);
static_assert(v == 55)
```

### constexpr模板常量

**常量别名**：

```c++
template<class T>
constexpr bool is_class_v = std::is_class<T>::value;
```

**表达式计算**：

```c++
template<char c>
constexpr bool is_digit = (c >= '0' && c <= '9');
template<char c>
constexpr bool is_digit_or_dot = (is_digit<c> || c == '.');

static_assert(! is_digit<'x'>);
static_assert(is_digit<'0'>);
```

**模板特化**：

```c++
template<size_t N>
constexpr size_t fibonacci = fibonacci<N - 1> + fibonacci<N - 2>;
template<>
constexpr size_t fibonacci<0> = 0;
template<>
constexpr size_t fibonacci<1> = 1;

static_assert(fibonacci<10> == 55);
```

### if constexpr

if constexpr能够在编译期对constexpr表达式进行条件判断。具体来说，在编译期间，编译器会发现有一个分支确定不会被执行（相当于 if(false) {}），那么这个分支就不会进行编译，直接剔除。

这个特性能够帮助我们对代码进行优化甚至是进行修改。比如，某个平台下完全不支持某个功能，那么这个功能就不应该被编译，过去我们可能依靠宏或者模板的SFINAE特性来帮助我们实现。

```c++
// 依靠宏
#ifdef XXX
  // ...
#else
  // ...
#endif

// 依靠模板SFINAE
// 仅当调用cond为真时，实例化该模板
template<T>
typename enable_if<cond, bool>::type func(){
  // ...
}
```

在有了if constexpr语法之后，我们可以配合着类型萃取或约束（C++20特性）来达成同样的效果，比如：

```c++
template<typename T, typename Tag>
void process(T t, Tag tag) {
  if constexpr(std::is_same_v<Tag, int>) {
    // 处理int标签版本
  } else {
    // 其他标签版本 
  }
}
```

通过这种方法实现跨平台支持，也被称为tag dispatch。

## 编译期运算

### 数值计算

除了使用constexpr函数在编译期进行运算之外，我们同样可以使用模板进行运算。比如，下面就是一个使用模板计算斐波那契数列的方法。

```c++
template <size_t N> 
struct Fibonacci {  
    constexpr static size_t value = 
        Fibonacci<N - 1>::value +
        Fibonacci<N - 2>::value;
};

template <> struct Fibonacci<0> {   
    constexpr static size_t value = 0;
};

template <> struct Fibonacci<1> {   
    constexpr static size_t value = 1;
}

template<size_t N>
constexpr size_t Fibonacci_v = Fibonacci<N>::value; 
```

### 类型计算

更常见的情况是使用模板进行类型计算，比如下面是一个计算类型T的指针类型的例子。

```c++
template <typename T> 
struct AddPointer {   
    using type = T*;  
};

template <typename T>
using AddPointer_t =
    typename AddPointer<T>::type; 
```

同样的我们也可以使用constexpr进行类型计算，比如使用constexpr if代替enable_if

```c++
// 使用enable_if
template<typename T>
std::enable_if_t<std::is_integral<T>::value, std::string>
to_string(T t)
{
  return std::to_string(t);
}

template<typename T>
std::enable_if_t<!std::is_integral<T>::value, std::string>
to_string(T t)
{
  return t;
}

// 使用constexpr if
template<typename T>
auto to_string(T t)
{
  if constexpr(std::is_integral<T>::value)
    return std::to_string(t);
  else
    return t;
}
```

### 编译时多态

奇异递归模板模式（Curiously Recurring Template Pattern，CRTP）是C++模板编程时的一种惯用法，它能够实现编译期的多态。其具体做法就是把派生类作为基类的模板参数。比如：

```c++
template <class T> 
struct Base
{
    void interface()
    {
        // ...
        static_cast<T*>(this)->implementation();
        // ...
    }

    static void static_func()
    {
        // ...
        T::static_sub_func();
        // ...
    }
};

struct Derived : Base<Derived>
{
    void implementation();
    static void static_sub_func();
};
```

在这个例子中，`Base<Derived>::interface()`，虽然是在`struct Derived`之前就被声明了，但未被编译器实例化直至它被实际调用，这发生于`Derived`声明之后，此时`Derived::implementation()`的声明是已知的。

这种技术获得了类似于虚函数的效果，并避免了动态多态的代价。因此CRTP被称为编译期多态。

### 静态反射

**反射是什么**：

The term reflection refers to the ability of a computer program to observe and possibly alter its own structure and/or its behavior. This includes building new or altering the existing data structures, doing changes to algorithms or changing the way the program code is interpreted. To some degree, reflective programming can be regarded a particular kind of metaprogramming.

**反射分类**：

Reflection can be classified as static/compile-time reflection and dynamic/run-time reflection. The static reflection means meta info about the program only available at complie-time, while the dynamic reflection can be queried when program is running.

**反射的作用**：

Common purposes of reflection include:

- serialization/store object data in JSON or XML;
- remote procedure calls;
- automatic generation of a relational schema from the application object model and
object-relational mapping;
- etc.

**Java的反射**：

Althogh the implementation of Java reflection is based on .class files which are generated at compile time, java reflection is dynamic, allowing it to inspect and possibly modify its structure while it is running. This is beacause JVM loads these .class files and provides reflection capabilities to inspect and interact with the bytecode at run time.

Relection in Java is widely used. For example, the two features of the most famous Java framework "Spring": IoC and AOP involve reflection. Other use scenarios of Java reflection include:

- dynamic proxies;
- testing frameworks;
- serialization mechanisms.

**C++原生的“反射”**：

Comparing to Java, C++ only supports *limited* compile-time reflection(also known as RTTI), primarily through the use of the typeid operator and dynamic_cast. Although the usefulness of RTTI mechanisms is quite limited, it merely costs nothing. It is actually a tradeoff between the complexity of the reflection system and its usefulness.

You don't pay for what you don't use. That's one of the must basic design philosophies underlying C++. That is why C++ only supports limited compile-time reflection.

## 参考

- [Modern C++模板元编程](https://netcan.github.io/presentation/metaprogramming/#/)
- [Constexpr 从11到20](https://netcan.github.io/presentation/constexpr_from_11_20/#/)
- [CppCon Constexpr all the things!](https://github.com/CppCon/CppCon2017/blob/master/Presentations/constexpr%20ALL%20the%20things/constexpr%20ALL%20the%20things%20-%20Jason%20Turner%20and%20Ben%20Deane%20-%20CppCon%202017.pdf)
- [如何优雅的实现 C++ 编译期静态反射](https://netcan.github.io/2020/08/01/%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E5%AE%9E%E7%8E%B0C-%E7%BC%96%E8%AF%91%E6%9C%9F%E9%9D%99%E6%80%81%E5%8F%8D%E5%B0%84/)
