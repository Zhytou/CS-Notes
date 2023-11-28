# 元编程

- [元编程](#元编程)
  - [模板起源](#模板起源)
    - [宏与泛型](#宏与泛型)
    - [零成本抽象](#零成本抽象)
  - [编译期运算](#编译期运算)
    - [数值计算](#数值计算)
    - [类型运算](#类型运算)

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

## 模板起源

模板起初是为了支持泛型替代而设计的语法。它的原理和宏类似，在编译期根据实际调用时传入的类型参数替换占位符，生成专属该类型的代码。但和宏相比，它具有类型安全、编译期检查错误以及零成本抽象的特点。

### 宏与泛型

在模板语法还未出现之时，依靠宏来实现泛型容器的方法如下：

```c++
// collection.h
struct Collection_ ## TYPE {
    TYPE *array; 
    size_t size, n;
};

#ifdef INSTANCE 
Collection_ ## TYPE make_Collection_ ## TYPE(size_t sz) {
    // ...
}
#endif

// main.cpp
#define INSTANCE

#define TYPE int 
#include "collection.h"
#undef TYPE

#define TYPE double
#include "collection.h"
#undef TYPE

int main() {
    Collection_int lstInt = make_Collection_int(5); 
    Collection_double lstDouble = make_Collection_double(5);
}
```

利用宏，实现了对collection.h中代码的复用。

### 零成本抽象

> C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

C++模板实现零成本抽象是因为模板只会根据实际调用时传入的类型参数替换占位符，生成专属该类型的代码。如果没有该类型的调用，则不会发生。

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

### 类型运算

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
