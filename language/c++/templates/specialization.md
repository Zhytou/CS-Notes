# 特化

- [特化](#特化)
  - [全特化](#全特化)
  - [偏特化](#偏特化)
  - [特化和萃取](#特化和萃取)

模板机制为C++提供了泛型编程的方式，在减少代码冗余的同时仍然可以提供类型安全。 特化必须在同一命名空间下进行，可以特化类模板也可以特化函数模板，但类模板可以偏特化和全特化，而函数模板只能全特化。模板实例化时会优先匹配"模板参数"最相符的那个特化版本。

## 全特化

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

## 偏特化

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

## 特化和萃取

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

**声明内嵌**：

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

**偏特化+萃取**：

所谓的`Traits`技术，又被叫做特性萃取技术。例如，下面这个模板就是使用了`Traits`技术来萃取迭代器的特性。

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
