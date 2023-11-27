# 模板编程

- [模板编程](#模板编程)
  - [定义模板](#定义模板)
    - [函数模板](#函数模板)
    - [类模板](#类模板)
  - [可变参数模板](#可变参数模板)
    - [可变参数函数模板](#可变参数函数模板)
    - [包扩展](#包扩展)
    - [转发包参数](#转发包参数)
  - [参考](#参考)

**模板编程 vs 元编程 vs 泛型编程**：

- 模板编程：指使用C++模板来实现通用和复用算法，在编译期生成对应类型的代码。
- 元编程：更广泛的概念包括任何可以操作或扩展编程语言本身的技术。模板编程属于其中一种形式。
- 泛型编程：面向接口编程思想，通过抽象化接口实现代码的重复利用性。C++模板是一种实现泛型编程的工具，但不等同于泛型编程这一概念。

## 定义模板

**typename vs class**:

在模板定义语法中关键字`class`与`typename`的作用完全一样。

但`typename`还有另外一个作用。它能表示使用`嵌套依赖类型(nested depended name)`。例如：

``` c++
 1 template <class I>
 2 struct iterator_traits {
 3     typedef typename I::value_type value_type;
 4 };
 5 
 6 template <class I>
 7 struct iterator_traits<T*> {
 8     typedef T value_type;
 9 };
```

### 函数模板

### 类模板

**类模板与继承**：

## 可变参数模板

一个`可变参数模板(variadic template)`就是一个接受可变数目参数的模板函数或者模板类。

可变数目的参数被称为`参数包(parameter packet)`。我们用一个省略号来指出一个可变数目参数。例如：

``` c++
template<typename T, typename... Args>
void func(const T& t, const Args& ... rest);
```

当我们想知道参数包中有多少参数时，我们可以使用`sizeof...()`运算符计算。

### 可变参数函数模板

可变参数函数通常是递归的。例如：

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

### 包扩展

对于一个参数包，除了能获取其大小外，我们能对它做的唯一事情就是`扩展(expand)`它。

扩展一个包就是将它分解为构成元素，对每个元素应用模式，获得扩展后的列表。我们通过在模式右边放一个省略号来触发扩展操作。

例如，下面的print函数包含了两个扩展：

``` c++
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args... rest) { // 扩展
  os << t;
  print(os, rest);// 扩展
}
```

### 转发包参数

我们可以使用可变参数模板与`std::forward`机制来编写函数，实现将其实参不变地转递给其他函数。例如：

``` c++
class StrVec{
  public:
    template<typename... Args> void func1(Args&&... args) {
      func2(std::forward<Args>(args));
    }
};
```

## 参考

- [C++ Primer](https://github.com/bumzy/book/blob/master/C%2B%2B%20%20Primer%E4%B8%AD%E6%96%87%E7%89%88%EF%BC%88%E7%AC%AC%E4%BA%94%E7%89%88%EF%BC%89.pdf)
- [STL 源码剖析-3.4 Traits编程技法](https://leezw.net/assets/pdf/STL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.pdf)
- [C++ Traits技术浅谈](https://www.cnblogs.com/mangoyuan/p/6446046.html)
