# 模板和泛型编程

- [模板和泛型编程](#模板和泛型编程)
  - [定义模板](#定义模板)
    - [函数模板](#函数模板)
    - [类模板](#类模板)
  - [模板参数推断](#模板参数推断)
    - [const转换规则](#const转换规则)
    - [右值引用](#右值引用)
    - [模板实参推断](#模板实参推断)
      - [从左值引用函数参数推断类型](#从左值引用函数参数推断类型)
      - [从右值引用函数参数推断类型](#从右值引用函数参数推断类型)
    - [引用折叠和右值引用参数](#引用折叠和右值引用参数)
      - [引子](#引子)
      - [第一种规则](#第一种规则)
      - [第二种规则——引用折叠](#第二种规则引用折叠)
    - [std::move vs std::forward](#stdmove-vs-stdforward)
      - [std::move](#stdmove)
      - [std::forward](#stdforward)
  - [Traits 技术](#traits-技术)
    - [模板参数推导](#模板参数推导)
    - [声明内嵌](#声明内嵌)
    - [偏特化](#偏特化)
  - [参考](#参考)

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

## 模板参数推断

### const转换规则

可以将一个非`const`对象的引用（或指针）传递给一个`const`的引用（或指针）形参。

### 右值引用

为了支持移动操作，新标准引入了一种新的引用类型——右值引用。

- 所谓的右值引用就是必须绑定到右值的引用。

  > 左值和右值是表达式的属性。
  >
  > 一般而言，一个左值表达式表示的是一个对象的省份，而一个右值表达式表示的是对象的值。

- 我们通过 && 而不是 & 来获取右值引用。

- 右值引用有一个重要的性质，只能绑定到一个将要销毁的对象。

类似任何引用，一个右值引用也不过是某个对象的另一个名字而已。

对于常规引用（为了和右值引用区分开来，我们可以将其称之为左值引用），我们不能将其绑定到要求转化的表达式、字面常量或是返回右值的表达式。

### 模板实参推断

#### 从左值引用函数参数推断类型

**非const型**

``` c++
template <typename T> void f(T&);//实参必须是一个左值
f(i);//i是一个int，模板参数T是int
f(ci);//ci是一个int，模板参数T是const int
f(5);//错误，实参必须是左值
```

**const型**

``` c++
template <typename T> void f2(const T&);//可以接收一个右值
f2(i);//i是一个int，模板参数T是int
f2(ci);//ci是一个const int，但模板参数T是int
f2(5);//一个const& 参数可以绑定到一个右值，模板参数T是int
```

#### 从右值引用函数参数推断类型

推断过程类似左值引用函数。

``` c++
template <typename T> void f3(T&&);
f3(42); // 实参是一个int类型的右值，模板参数T是int
```

### 引用折叠和右值引用参数

#### 引子

``` c++
template <typename T> void f3(T&&);
f3(i);// 正确，因为引用迭代
```

看上述例子，`i`作为一个int对象，理论上`f3(i)`这样的调用应该不合法。毕竟，`i`是一个左值，而通常我们不能将一个右值引用绑定到一个左值上去。

但C++定义了两种特殊的规则，允许这种绑定。此外，这两个例外的规则还是`std::move`这种标准库设施的工作基础。

#### 第一种规则

第一例外规则影响右值引用参数的推断进行。

当我们将一个左值传递给右值引用参数，且此右值引用指向模板类型参数（如T&&）时，编译器推断模板类型参数为实参的左值引用类型。

因此当我们调用`f3(i)`时，编译器推断T的类型为int&，而非int。

#### 第二种规则——引用折叠

在这种情况下，我们可以使用使用第二个例外的绑定规则：

如果我们简介创建了一个引用的引用，则这些引用形成了“折叠”。在这种情况下（一个例外），引用会折叠成以恶搞普通的左值引用。即，对于一个给定类型X：

+ X& &、X& &&和X&& &都折叠成类型&；
+ 类型X&& &&折叠成X&&。

在这种规则下，我们将得到一个惊人的结果，即：如果一个函数参数是指向模板参数类型的右值引用，则可以传递给他任意类型的实参。

### std::move vs std::forward

#### std::move

**描述**

我们还可以通过调用一个名为`std::move`的新标准函数来获得绑定到左值上的右值引用。

我们必须认识到，调用`move`就意味着承诺，除了对这个左值引用赋值或销毁它外，我们将不再使用它。

#### std::forward

**描述**

C++标准库定义了`std::forward`函数来保证实现完美转发。

其中，完美转发是指参数在传递过程中保持其值属性的功能，即若是左值，则传递之后仍然是左值，若是右值，则传递之后仍然是右值。

而`std::forward`的功能是：

当参数不是一个左值引用时，函数返回一个右值引用；

当参数是一个左值引用时，函数什么都不修改直接返回。

## Traits 技术

### 模板参数推导

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

### 声明内嵌

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

### 偏特化

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

## 参考

- [C++ Primer](https://github.com/bumzy/book/blob/master/C%2B%2B%20%20Primer%E4%B8%AD%E6%96%87%E7%89%88%EF%BC%88%E7%AC%AC%E4%BA%94%E7%89%88%EF%BC%89.pdf)
- [STL 源码剖析-3.4 Traits编程技法](\https://leezw.net/assets/pdf/STL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.pdf)
- [C++ Traits技术浅谈](https://www.cnblogs.com/mangoyuan/p/6446046.html)
