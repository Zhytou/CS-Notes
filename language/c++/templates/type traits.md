# 类型萃取

- [类型萃取](#类型萃取)
  - [标准库和萃取](#标准库和萃取)
  - [萃取的标准库支持](#萃取的标准库支持)
    - [最基本的萃取](#最基本的萃取)
    - [类型判断](#类型判断)
    - [根据条件选择萃取](#根据条件选择萃取)
    - [根据条件启用或禁用萃取](#根据条件启用或禁用萃取)

类型萃取，也被称为类型提取，是指将类模板的一部分接口提取出来，形成一个新的独立的模板的技术。

## 标准库和萃取

标准库算法和容器是分离的，两者通过迭代器链接。算法的实现并不知道自己被传进来什么。

## 萃取的标准库支持

C++11提供了type_traits库，来支持在编译期间计算、判断、转换或查询等功能。

### 最基本的萃取

std::integral_constant是标准库中最简单基础的萃取工具类。它的作用是获取一个编译器常量，其实现方法如下：

```c++
template<class T, T v>
struct integral_constant
{
    static constexpr T value = v;
    using value_type = T;
    using type = integral_constant; // using injected-class-name
    constexpr operator value_type() const noexcept { return value; }
    constexpr value_type operator()() const noexcept { return value; } // since c++14
};
```

### 类型判断

标准库还提供了一些用于判断类型工具，它们都是从std::integral_constant派生而来的。常见的包括：

- std::is_void
- std::is_integral
- std::is_floating_point
- std::is_array
- std::is_pointer
- std::is_enum
- std::is_union
- std::is_class

此外，还有一些用于类型关系判断模板：

- std::is_same：判断类型是否相同；
- std::is_base_of：判断Base类型是否为Derived类型的基类；
- std::is_convertible：判断前面的模板参数类型能否转化为后面的模板参数类型。

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
