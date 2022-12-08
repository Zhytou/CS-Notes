# 操作重载与类型转换

- [操作重载与类型转换](#操作重载与类型转换)
  - [下标运算符](#下标运算符)
  - [递增或递减运算符](#递增或递减运算符)
  - [函数调用运算符（functor）](#函数调用运算符functor)
    - [lambda时函数对象](#lambda时函数对象)
    - [标准库定义的函数对象](#标准库定义的函数对象)
    - [可调用对象与std::function](#可调用对象与stdfunction)
  - [重载、类型转换与运算符](#重载类型转换与运算符)

## 下标运算符

表示容器的类通常可以通过元素在容器中的位置访问元素，这些类一般会定义下标运算符`operator[]`。

下标运算符通常以所访问的元素引用作为返回值。此外，我们最好同时定义下标运算符的常量版本和非常量版本。

当作用到一个常量时，我们通常希望返回常量引用以确保不会给返回对象赋值。

``` c++
class StrVec{
public:
    const string& operator[](size_t n) const;
    string& operator[](size_t n);
}
```

## 递增或递减运算符

**如何区分前置和后置运算符**:

``` c++
class XXX{
  public:
  // 前置运算符 ++XXX
  XXX& operator++();
  // 后置运算符 XXX++
  XXX& operator++(size_t);

}
```

## 函数调用运算符（functor）

如果类重载了函数调用运算符，则我们可以像使用函数一样使用该类对象。由于这样的类也能同时存储状态，所以比普通函数更加灵活。

### lambda时函数对象

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

### 标准库定义的函数对象

标准库定义了一组表示算数运算符、关系运算符和逻辑运算符的类。这些类都被定义成模板的形式存放在`functional`头文件中。

|                算术             |              关系           |    逻辑      |
|---------------------------------|----------------------------|--------------|
|`plus<Type>` `minus<Type>`           | `less<Type>` `greater<Type>`   |

### 可调用对象与std::function

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

## 重载、类型转换与运算符

类型转换运算符是类的一种特殊成员函数，它负责将一个类类型的值转换成其他类型，一般定义成`explicit`避免异常情况发生。其形式如下：

`explicit operator type() const;`
