# C++ Standards

- [C++ Standards](#c-standards)
  - [C++11](#c11)
    - [constexpr](#constexpr)
    - [auto and decltype](#auto-and-decltype)
    - [final and override](#final-and-override)
    - [lambda](#lambda)
    - [smart pointer](#smart-pointer)
    - [std::initializer\_list](#stdinitializer_list)
    - [类内初始值](#类内初始值)
    - [列表初始化](#列表初始化)
    - [列表初始化返回值](#列表初始化返回值)
  - [C++14](#c14)
    - [generic lambda](#generic-lambda)
    - [std::make\_unique](#stdmake_unique)
  - [C++17](#c17)
    - [if constexpr](#if-constexpr)
    - [std::optional](#stdoptional)
    - [std::filesystem](#stdfilesystem)
  - [C++20](#c20)
    - [std::concept](#stdconcept)

## C++11

### constexpr

constexpr用于声明可在编译时求值的常量表达式，其值在编译时确定。

**变量与constexpr**：

对于constexpr变量来说，必须用字面量或者constexpr函数初始化。

**函数与constexpr**：

constexpr函数可以用于计算常量表达式。但它也要求函数体中的操作必须是可求值的常量表达式，且不能包含具有副作用的操作（如 I/O 操作），并且参数类型和返回类型必须满足一定的要求。此外，编译器通常会尝试对 constexpr 函数进行内联展开，将函数调用替换为函数体的内容，以减少函数调用的开销。

**构造函数与constexpr**：

字面值类型的构造函数常常用constexpr修饰，比如：

```c++
struct Literal {
  constexpr Literal(int x, int y) : m_x(x), m_y(y) {}
  int m_x, m_y;
};

Literal point = Literal(1, 2); // 合法,在编译期构造对象
```

**const与constexpr**：

其实，const并不能代表“常量”，它仅仅是对变量的一个修饰，告诉编译器这个变量只能被初始化，且不能被直接修改（实际上可以通过堆栈溢出等方式修改）。而这个变量的值，可以在运行时也可以在编译时指定。

constexpr可以用来修饰变量、函数、构造函数。一旦以上任何元素被constexpr修饰，那么等于说是告诉编译器 “请大胆地将我看成编译时就能得出常量值的表达式去优化我”。

### auto and decltype

**auto**是C++11引入的关键字，用于自动推断变量的类型。准确来说，auto在C++98时就已经存在，只不过作用是声明变量拥有自动生命周期。C++11更改了它的作用。当使用auto声明变量时，编译器会根据变量的初始化表达式来推断其类型。例如：

```c++
// 推断x为int
auto x = 5;
```

**decltype**同样是C++11引入的关键字，用于获取表达式的类型。它可以在编译时获取表达式的静态类型，并将其作为变量的类型或函数返回类型。比如：

```c++
int x = 5;
decltype(x) y;  // y 的类型为 int
```

除此之外，decltype还常常用在获取lambda表达式类型上，比如定义一个优先队列：

```c++
auto cmp = [](pair<int, int> a, pair<int, int>) {
  return a.first > b.frist;
};
priority_queue<pair<int,int>, vector<pair<int, int>>, decltype(cmp)> pq(cmp);
```

### final and override

C++11中引入了final和override两个关键字，用于控制类的继承和虚函数的覆写行为。

**override**：

C++11允许程序员使用`override`关键字来说明派生类中的虚函数。这么做的好处是使得程序员的意图更加清晰，同时让编译器为我们发现一些错误。

如果我们使用了`override`标记了某个函数，但该函数没有覆盖已经存在的虚函数，此时编译器会报错。因为如果派生类定义了一个普通（或虚）函数与基类中函数的名字相同但形参列表不同，这仍然是合法的。

**final**：

C++11新标准提供了一种防止继承发生的方法，即关键字`final`。

- 修饰类，让类无法被继承。
- 修饰类的虚函数，让虚函数在子类中不能被重写。

注意，final只能修饰类与虚函数，无法修饰普通成员函数。

### lambda

一个lambda表达式一般具有如下形式：
  
```C++
[capture list](parameter list) -> return type { function body }
```  

其中，捕获列表是一个lambda所在函数中定义的局部变量列表，它表示lambda函数体中可以使用这些局部变量。此外，非局部变量，如静态变量、全局变量等可以不经捕获，直接使用。

一般来说，捕获列表可分为按值捕获、引用捕获以及隐式捕获。三者区别如下所示。

```c++
int a = 1;
// 按值捕获
auto f1 = [a]() {

};
// 引用捕获
auto f2 = [&a]() {

};
// 隐式捕获，使用&指定引用捕获全部变量；使用=指定值捕获全部变量
auto f3 = [=]() {

};
```

### smart pointer

为了更安全地使用动态内存，C++11标准提供智能指针来管理动态对象，关于这一部分的介绍可以查看[笔记-智能指针](https://github.com/Zhytou/CS-Notes/blob/main/language/c%2B%2B/memory%20management/smart%20pointers.md)。

### std::initializer_list

- 为了能编写能处理不同实参的函数，C++11 新标准提供了两种主要的方法：
  - ==如果所有实参类型相同==，可以传递一个名为initializer_list的标准库类；
  - 如果实参类型不同，我们可以编写一种特殊函数，也就是所谓的可变参数模板。
- initializer_list是一种标准库类型，用于表示某种特定类型的值的数组。initializer_list就放在名为initializer_list的库内。
  - 定义initializer_list必须说明所含元素类型；
  - initializer_list对象中的元素永远是常量值；
  - 含有initializer_list形参的函数也可以同时拥有其他形参。

### 类内初始值

C++11新标准规定，可以为数据成员提供一个类内初始值。当创建对象时，类内初始值将用于初始化数据成员，比如：

```c++
struct A{
  int i = 100;
  char ch = 'a';
  double d = 0.01;
};

int main() {
  A a;
  // 100a0.01
  cout << a.i << a.ch << a.d << endl;
  return 0;
}
```

### 列表初始化

C++11新标准提供了一种列表初始化的方法。

### 列表初始化返回值

在C++11之前，如果我们想要返回一组数据，我们必须在子函数中构造一个对应的容器，借助容器来进行返回。但C++11的新标准下，我们可以直接返回字面值，该字面值会用于容器的构造，而无需我们自己去构造。

```c++
vector<int> process()
{
 return {1,2,3,4};
}
```

## C++14

### generic lambda

所谓泛型lambda，就是在形参声明中使用auto类型指示说明符的lambda。比如：

```c++
auto lambda = [](auto x, auto y) {return x + y;};

// 在C++14标准下，上述lambda表达式与以下代码作用相同。
struct unnamed_lambda
{
  template<typename T, typename U>
    auto operator()(T x, U y) const {return x + y;}
};
auto lambda = unnamed_lambda();
```

### std::make_unique

C++11时，只给shared_ptr指针提供了make_shared函数，却没有给unique_ptr指针提供make_unique函数。C++14改善了这一点。

## C++17

### if constexpr

### std::optional

### std::filesystem

## C++20

### std::concept
