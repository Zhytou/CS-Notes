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
    - [类型萃取](#类型萃取)
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

### 特化和SFINAE

SFINAE规则，即替换失败并非错误（Substitution Failuer Is Not An Error）。它是指C++语言在模板参数匹配失败时不认为这是一个编译错误。这种特性就允许C++程序根据条件启用或禁用萃取。

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

### 类型萃取

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

### 静态反射

## 参考

- [Modern C++模板元编程](https://netcan.github.io/presentation/metaprogramming/#/)
- [Constexpr 从11到20](https://netcan.github.io/presentation/constexpr_from_11_20/#/)
- [CppCon Constexpr all the things!](https://github.com/CppCon/CppCon2017/blob/master/Presentations/constexpr%20ALL%20the%20things/constexpr%20ALL%20the%20things%20-%20Jason%20Turner%20and%20Ben%20Deane%20-%20CppCon%202017.pdf)
