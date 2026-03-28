# 类型萃取

- [类型萃取](#类型萃取)
  - [为什么需要类型萃取器](#为什么需要类型萃取器)
    - [声明内嵌：一种简单的萃取器实现方法](#声明内嵌一种简单的萃取器实现方法)
    - [偏特化和萃取器配合](#偏特化和萃取器配合)
    - [萃取器的核心：编译期类型与值的封装](#萃取器的核心编译期类型与值的封装)
    - [萃取器的替代品：嵌套template参数](#萃取器的替代品嵌套template参数)
  - [萃取的标准库支持](#萃取的标准库支持)
    - [最基本的萃取器](#最基本的萃取器)
    - [常见萃取器](#常见萃取器)
    - [根据条件选择萃取](#根据条件选择萃取)
    - [根据条件启用或禁用萃取](#根据条件启用或禁用萃取)

类型萃取（Type Traits），是一种模板元编程技术。其核心在于：在编译期提取、查询或修改类型的属性。

## 为什么需要类型萃取器

由于C++是一种静态类型语言，模板函数在处理未知类型T时，就像在黑盒里操作。类型萃取允许我们在不运行程序的情况下，提前获知T是不是指针、有没有构造函数，甚至强行把它的const属性去掉，为开发提供了更多的灵活性。

比如，在实现迭代器时，我们可以使用模板参数推导机制写出如下代码：

``` c++
template <typename Iter, typename ElementType>
void func_impl(Iter iter, ElementType type) {
}

template <typename Iter>
void func_wrapper(Iter iter) {
  func_impl(iter, *iter); 
}
```

这里我们需要多写一个包装函数来掩盖底层实现，否则就要每次多传一个迭代器指向的值，而且无法返回指向类型。显然，我们还需要找到更好的方法。

### 声明内嵌：一种简单的萃取器实现方法

为了直接获取迭代器指向元素类型，一种可行做法便是在迭代器类中内嵌一个`value_type`的类型声明：

``` c++
template <typename T>
struct MyIter {
  typedef T value_type; // 内嵌型别声明
};

template <typename I>
// 返回类型为迭代器指向类型
typename I::value_type func(I itr) {
  // ... 函数功能主体
  return *itr;
}
```

这种方式对自定义迭代器类有效，逻辑清晰，也符合“将接口与实现分离”的设计原则。然而，它存在一个根本性局限：无法支持原生指针（如 int*）。

因为原生指针是语言内置类型，无法为其添加`::value_type`成员。一旦该指针传入函数，编译器便将因找不到`int*::value_type`而报错。

### 偏特化和萃取器配合

针对原生指针的情况，便需要偏特化对迭代器模板进行定制化的实现

``` c++
// 迭代器萃取器
template<typename I>
struct iterator_traits {
  typedef typename I::value_type value_type;
}

// 该迭代器萃取器 偏特化版本（支持原生指针）
template<typename T>
struct iterator_traits<T*>{
  typedef T value_type;
}

// 某迭代器函数（支持原生指针）
template<typename I>
typename iterator_traits<I>::value_type func(I itr) {}
```

### 萃取器的核心：编译期类型与值的封装

**核心成员**：

在C++模板元编程中，萃取器的本质是一种在编译期封装类型信息或常量值的机制。其典型实现通常包含两个核心成员：

- value_type：一个类型别名（using 或 typedef），用于表示所关联的类型（如 bool、int 等）；
- value：一个 static constexpr 静态常量，用于存储具体的编译期布尔值或整型常量。

不仅如此，为了方便使用还会定义缩写_t以及_v的缩写，比如：

```c++
template<typename T, bool v>
struct example_trait {
  using value_type = T;
  static constexpr bool value = v;
  // ...
};

// 定义缩写
template <typename T, bool v>
using example_trait_t = typename example_trait<T, v>::value_type;
template <typename T, bool v>
constexpr bool example_trait_v = example_trait<T, v>::value;

// 使用对比
typename example_trait<int, true>::value_type a; // 旧
example_trait_t<int, true> a;                   // 新，清爽很多
if (example_trait<int, true>::value) { ... } // 旧
if (example_trait_v<int, true>) { ... }      // 新
```

这种设计使得萃取器既能作为类型载体，又能作为编译期常量表达式参与条件判断、模板特化和 SFINAE 等元编程技术。

**编译期常量**：

其中，在C++11引入constexpr关键字之前，想要进行编译期计算，只能依靠模板参数以及enum关键字实现。比如：

```c++
template <int N>
struct Fibonacci {
  // 在 C++11 之前，这是定义“编译期变量”的标准写法
  enum { value = Fibonacci<N - 1>::value + Fibonacci<int N - 2>::value };
};

template <>
struct Fibonacci<0> {
  enum { value = 0 };
};

template <>
struct Fibonacci<1> {
  enum { value = 1 };
};

```

### 萃取器的替代品：嵌套template参数

除了萃取器外，一种常见类型查询或修改的方法是使用嵌套template参数。比如：

``` c++
template <template<class, class> class V, class T, class A>
void f(V<T, A> &v) {
  // This can be "typename V<T, A>::value_type",
  // but we are pretending we don't have it
}
```

不过，相比萃取器，嵌套template参数的做法并没有完全解耦，且模板固化（比如上述例子假定了容器正好有两个模板参数）。

## 萃取的标准库支持

C++11提供了type_traits库，来支持在编译期间计算、判断、转换或查询等功能。

### 最基本的萃取器

std::integral_constant是标准库中最简单基础的萃取工具类。它的作用是获取一个编译器常量，其实现方法如下：

```c++
template<typename T, T v>
struct integral_constant {
  static constexpr T value = v;
  using value_type = T;
  using type = integral_constant; // using injected-class-name
  constexpr operator value_type() const noexcept { return value; }
  constexpr value_type operator()() const noexcept { return value; } // since c++14
};
```

使用std::integral_constant我们可以定义true和false两个布尔类型常量。

```c++
template <bool _Val>
using bool_constant = integral_constant<bool, _Val>;

using true_type  = bool_constant<true>;
using false_type = bool_constant<false>;
```

利用true_type和false_type，我们可以定义一系列类型判断萃取器，比如用于判断类型是否为整型的is_integral：

```c++
template<typename> struct is_integral_base: std::false_type {};

// 全特化出所有整型，包括：bool、char、short、int、long等等
template<> struct is_integral_base<bool>: std::true_type {};
template<> struct is_integral_base<int>: std::true_type {};
template<> struct is_integral_base<short>: std::true_type {};
// ...

// std::remove_cv除去类型的const和volatile修饰符
template<typename T> struct is_integral: is_integral_base<std::remove_cv_t<T>> {};
```

使用时，直接依靠其value成员判断true/false即可：

```c++
template<typename T>
void process(T x) {
  if constexpr (is_integral_v<T>) {
    std::cout << "Integral: " << x << '\n';
  } else {
    std::cout << "Non-integral\n";
  }
}
```

### 常见萃取器

标准库还提供了一些用于判断类型工具，它们都是从std::integral_constant派生而来的。常见的包括：

**Type Categories**:

- std::is_void：判断类型是否为空；
- std::is_integral：判断类型是否为整型；
- std::is_floating_point：判断类型是否为浮点数；
- std::is_pointer：判读类型是否为指针；
- std::is_class：判断类型是否为类。

**Type Properties**:

- std::is_const
- std::is_volatile
- std::is_empty

**Relationships and Property Queries**:

- std::is_same：判断类型是否相同；
- std::is_base_of：判断Base类型是否为Derived类型的基类；
- std::is_convertible：判断前面的模板参数类型能否转化为后面的模板参数类型。

**Type Modifications**:

- std::remove_cv
- std::remove_const
- std::remove_volatile

**Type Transformation**:

- std::result_of
- std::invoke_result

### 根据条件选择萃取

std::conditional在编译期根据一个判断式选择两个类型中的一个，类似一个三元表达式。它的原型如下：

```c++
template <bool B, class T, class F>
struct conditional;
```

当B为真时，则std::conditional::type为T；反之，为F。

### 根据条件启用或禁用萃取

编译器在匹配重载函数时，会尝试匹配所有重载函数，找到一个最精准的匹配。

**SFINAE**：替换失败并非错误（Substitution Failuer Is Not An Error）就是指C++语言在模板参数匹配失败时不认为这是一个编译错误。这种特性就能允许根据条件启用或禁用萃取。

std::enable_if就是标准库利用SFINAE实现根据条件选择重载函数，其的原型如下：

```c++
template<bool B, class T = void>
struct enable_if;
```

std::enable_if使得函数在判断条件B为真时才有效。
