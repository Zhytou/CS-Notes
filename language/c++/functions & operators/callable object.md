# 可调用对象

- [可调用对象](#可调用对象)
  - [Lambda表达式](#lambda表达式)
  - [仿函数类 Functor class](#仿函数类-functor-class)
  - [lambda时函数对象](#lambda时函数对象)
  - [标准库定义的函数对象](#标准库定义的函数对象)
  - [可调用对象包装器——std::function](#可调用对象包装器stdfunction)

## Lambda表达式

A lambda expression creates an nameless functor, it's syntactic sugar.

一个`lambda`表达式可以表示一个可以调用的代码单元。我们可以将其理解为一个未命名的仿函数对象。

一般来说，一个`lambda`表达式具有如下形式`[capture list](parameter list) -> return type {function body}`。

其中，`capture list`是一个`lambda`表达式所在作用域中定义的局部变量列表，通常为空。 `lambda`表达式使用捕获列表来明确它将使用的变量。

`lambda`表达式可以使用值捕获或引用捕获的方式，具体使用方法如下：

``` c++
int sz;

// 值捕获sz
auto cmp1 = [sz](const string& s) {
  return s.size() >= sz;
}

// 引用捕获sz
auto cmp2 = [&sz](const string& s) {
  return s.size() >= sz;
}

// 隐式值捕获（捕获所有局部变量）
auto cmp3 = [=](const string& s) {
  return s.size() >= sz;
}

// 隐式引用捕获（捕获所有局部变量）
auto cmp4 = [&](const string& s) {
  return s.size() >= sz;
}
```

补充一些更复杂的捕获列表及其含义：

- a, &b。将a按值进行传递，b按引用进行传递。
- =，&a, &b。除a和b按引用进行传递外，其他参数都按值进行传递。
- &, a, b。除a和b按值进行传递外，其他参数都按引用进行传递

另外， 如果我们想捕获一个类内变量，需要在捕获列表内传入`this`指针。例如：

``` c++
// 隐式引用捕获（捕获所有局部变量）
auto cmp5 = [this](const string& s) {
  return s.size() >= sz;
}
```

此外，如果我们希望改变一个被捕获的变量的值，就必须在参数列表加上关键字`mutable`。

## 仿函数类 Functor class

Functors are objects that can be treated as though they are a function or function pointer.

A functor is pretty much just a class which defines the operator(). That lets you create objects which "look like" a function

One nice thing about functors is that unlike regular functions, they can contain state which means that it can be customized.

如果类重载了函数调用运算符，则我们可以像使用函数一样使用该类对象。由于这样的类也能同时存储状态，所以比普通函数更加灵活。

**函数包装模板 std::function**:

std::function是一个可变参类模板，是一个通用的函数包装器（Polymorphic function wrapper）。std::function的实例可以存储、复制和调用任何可复制构造的可调用目标，包括普通函数、成员函数、类对象（重载了operator()的类的对象）、Lambda表达式等。是对C++现有的可调用实体的一种类型安全的包裹（相比而言，函数指针这种可调用实体，是类型不安全的）。

std::function中存储的可调用对象被称之为std::function的目标。若std::function中不含目标，调用不含目标的std::function会抛出std::bad_function_call 异常。

## lambda时函数对象

当我们编写以一个`lambda`后，编译器将该表达式翻译成一个未命名类的未命名对象。例如：

``` c++

auto cmp1 = [](const string& a, const string& b) {
  return a.size() < b.size();
}

class StrCmp1 {
  public:
    bool operator() (const string& a, const string& b) {
      return a.size() < b.size();
    }
}

int sz;

auto cmp2 = [&](const string& a) {
  return a.size() < sz;
}

class StrCmp2 {
  int sz;
  StrCmp2(int n) : sz(n) {} 
  public:
    bool operator() (const string& a) {
      return a.size() < sz;
    }
}
```

## 标准库定义的函数对象

标准库定义了一组表示算数运算符、关系运算符和逻辑运算符的类。这些类都被定义成模板的形式存放在`functional`头文件中。

|                算术             |              关系           |    逻辑      |
|---------------------------------|----------------------------|--------------|
|`plus<Type>` `minus<Type>`           | `less<Type>` `greater<Type>`   |

## 可调用对象包装器——std::function

C++语言中有几种可调用的对象：函数、函数指针、`lambda`表达式、`bind`创建的对象以及重载了函数调用运算符的类。

和其他对象一样，可调用的对象也有类型。例如：每个`lambda`表达式有它自己唯一的类类型。然而两个不同类型的可调用对象可能共享一种`调用形式(call signature)`。

`std::function`是一个类模板，一个特定类型的`function<T>`可以存储拥有这种调用形式的可调用对象。例如：

``` c++
int add(int a, int b) { return a + b; }

auto mod = [](int a, int b) { return a % b; };

class divide {
  public:
    int operator(int a, int b) {
      return a / b;
    }
};

// 这里我们定义了一个可以接受两个int、返回一个int的可调用对象
function<int(int, int)> f;

f = add;
f = mod;
f = divide;

// 用2和3当作输入参数调用该对象
f(2, 3);
```
