# constexpr 元编程

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

## 深入constexpr

### constexpr常量
