# 类型萃取

- [类型萃取](#类型萃取)
  - [类型萃取引入](#类型萃取引入)
    - [萃取器的实现——声明内嵌](#萃取器的实现声明内嵌)
    - [偏特化和萃取器配合](#偏特化和萃取器配合)
    - [嵌套template参数——萃取器的替代品](#嵌套template参数萃取器的替代品)
  - [萃取的标准库支持](#萃取的标准库支持)
    - [最基本的萃取](#最基本的萃取)
    - [常见萃取器](#常见萃取器)
    - [根据条件选择萃取](#根据条件选择萃取)
    - [根据条件启用或禁用萃取](#根据条件启用或禁用萃取)

类型萃取，也被称为类型提取，是指将类模板的一部分接口提取出来，形成一个新的独立的模板的技术。

## 类型萃取引入

在实现迭代器时，我们可以使用模板参数推导机制写出如下代码：

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

### 萃取器的实现——声明内嵌

声明内嵌型别似乎是个好主意，这样我们就可以直接获取元素类型。

``` c++
template <typename T>
class MyIter {
  typedef T value_type; // 内嵌型别声明
};

template <class I>
// 返回类型为迭代器指向类型
typename I::value_type func(I ite) {
  return *ite;
}
```

尽管上述代码已经很不错了，但它有一个致命的问题——不支持原生指针。这时候就需要偏特化的出现了。

### 偏特化和萃取器配合

通过偏特化，我们得以针对特定类型或特定条件，对模板进行定制化的实现

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

### 嵌套template参数——萃取器的替代品

``` c++
template <template<class, class> class V, class T, class A>
void f(V<T, A> &v) {
    // This can be "typename V<T, A>::value_type",
    // but we are pretending we don't have it
}
```

## 萃取的标准库支持

C++11提供了type_traits库，来支持在编译期间计算、判断、转换或查询等功能。

### 最基本的萃取

std::integral_constant是标准库中最简单基础的萃取工具类。它的作用是获取一个编译器常量，其实现方法如下：

```c++
template<class T, T v>
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
