# 类型萃取

- [类型萃取](#类型萃取)
  - [标准库支持](#标准库支持)
    - [最基本的萃取——定义编译器常量](#最基本的萃取定义编译器常量)
    - [类型判断](#类型判断)

## 标准库支持

C++11提供了type_traits库，来支持在编译期间计算、判断、转换或查询等功能。

### 最基本的萃取——定义编译器常量

std::integral_constant

**实现**：

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

- std::is_integral
- std::is_floating_point
- 
