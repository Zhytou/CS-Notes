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
    - [std::variant](#stdvariant)
    - [std::filesystem](#stdfilesystem)
    - [模板参数推导](#模板参数推导)
  - [C++20](#c20)
    - [concept](#concept)
    - [std::ranges](#stdranges)
    - [corountine](#corountine)

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

if constexpr是C++17 中引入的一个新特性，它是一种编译时条件判断语句。与传统的if语句不同，if constexpr在编译时根据条件表达式的结果选择是否编译其中的代码块，不符合条件的分支在编译时被丢弃，不会出现在最终的可执行代码中。这使得编译器能够进行更多的优化，并且在模板元编程中非常有用。

```c++
if constexpr (std::is_integral<T>::value) {
  // 当 T 是整数类型时，编译此代码块
  std::cout << "T is an integral type." << std::endl;
} else {
  // 当 T 不是整数类型时，编译此代码块
  std::cout << "T is not an integral type." << std::endl;
}
```

### std::optional

std::optional是C++17中引入的一个标准库模板类，用于表示一个可能包含值的容器。它的作用类似与Python中的None类似。

比如，编写一个查找数组中第一个非零元素的函数，如果没有std::optional类的话，可能我们使用返回一个pair来表示。其中，第一个位置表示是否包含非0元素，第二个位置表示非0元素的值。

```c++
int findFirstNonZero(vector<int>& arr) {
  for (int i = 0; i < arr.size(); i++) {
    if (arr[i] != 0) {
      return make_pair(true, arr[i]);
    }
  }
  return make_pair(false, -1);
}
```

采用std::optional可以简化上面的代码：

```c++
std::optional<int> findFirstNonZero(vector<int>& arr) {
  for (int i = 0; i < arr.size(); i++) {
    if (arr[i] != 0) {
      return arr[i];
    }
  }
  return std::nullopt;
}
```

此时，调用方只需使用has_value()和value()函数访问该optional即可，比如：

```c++
int main() {
  vector<int> nums = {0, 0, -1};
  auto ret = findFirstNonZero(nums);
  if (ret.has_value()) {
    cout << ret.value() << endl;
  }
}
```

### std::variant

std::variant是C++17中引入的另一个标准库模板类，用于表示一组可能类型中的一个。它的功能类似联合体Union，但其功能更强大，可以使用其他容器。比如

```c++
std::variant<int, double, std::string> var;
var = 42;
if (std::holds_alternative<int>(var)) {
  std::cout << "The variant holds an int: " << std::get<int>(var) << std::endl;
}
```

### std::filesystem

std::filesystem是C++17中引入的标准库，用于操作文件系统。它提供了一组类和函数，用于处理文件、目录和路径等文件系统相关的操作。

### 模板参数推导

C++17中添加了一个新特性，称为模板类参数推导。这样类似std::lock_guard这样简单的模板类型的模板参数列表可以省略。

```c++
std::mutex mtx;
std::lock_guard lck(mtx);
```

## C++20

### concept

concept是C++20中引入的一个关键字，用于定义对模板参数的一条约束。简单来说，可以将其看作是对原来使用enable_if和 constexpr等技术进行模板编程的一种改进。它提供了更直观和可读的语法，使得模板参数的约束更加清晰和简单。比如：

```c++
#include <iostream>
#include <concepts>
#include <type_traits>

// 使用enable_if实现
template <typename T>
typename std::enable_if_t<std::is_integral_v<T>, void>
printValue(T value) {
  std::cout << "Integral value: " << value << std::endl;
}

// 使用if constexpr实现
template <typename T>
void printValue(T value) {
  if constexpr(std::is_integral_v<T>) {
    std::cout << "Integral value: " << value << std::endl;
  }
}

// 使用concept实现
template <typename T>
requires std::integral<T>
void printValue(T value) {
  std::cout << "Integral value: " << value << std::endl;
}
```

从上面可以看出，依靠enable_if这类萃取器实现SFINAE和特化只能依靠返回值来限制，而if constexpr和concept则灵活的多，直接对传入类型进行限制即可。具体来说，使用requires关键字限制传入类型，有下面四种写法。

```c++
template <integral T>
T inc(T& a) { return ++a; }

integral auto inc(integral auto a) { return ++a; }

template <typename T>
T inc(T a) requires integral<T> { return ++a; }

template <typename T>
requires integral<T> 
T inc(T a) { return ++a; }
```

其中，integral是标准库定义的一种concept，实现如下：

```c++
template <typename T>
concept integral = std::is_integral<T>::value;
```

除了使用标准库预先定义好的concept之外，我们也可以使用下面两种形式进行自定义。

- requires { requirement-seq }
- requires ( parameter-list(optional) ) { requirement-seq }

下面是使用这两种形式进行自定义的实例。

```c++
template <typename T>
concept Addable = requires(T a, T b) { a + b; };  // a + b 可通过编译即可

template <class T>
concept Check = requires(T a, T b) {
  { a.clear() } noexcept;  // 支持clear,且不抛异常
  { a + b } noexcept->std::same_as<int>;  // std::same_as<decltype((a + b)), int>
};

template <typename T>
concept Check = requires(T x) {
  {*x};                                 // *x有意义
  { x + 1 } -> std::same_as<int>;       // x + 1有意义且std::same_as<decltype((x + 1)), int>，即x+1是int类型
  { x * 1 } -> std::convertible_to<T>;  // x * 1 有意义且std::convertible_to< decltype((x *1),T>，即x*1可转变为T类型
};
```

### std::ranges

std::ranges是C++20中引入的一个新的标准库组件，用于处理序列和容器。它提供了一组通用的操作函数，如过滤、映射、排序等，使得对序列的操作更加简洁和高效。具体来说，一个range是指任何拥有begin()和end()成员函数的类型。它实际是一个概念，具体实现如下：

```c++
template< class T >
concept range = requires( T& t ) {
  ranges::begin(t); // equality-preserving for forward iterators
  ranges::end (t);
};
```

这样，任何满足range概念的类型就可以被视为一个范围，进而可以直接使用std::ranges中的算法和操作符。其中，常见的用法包括：

```c++
std::vector<int> numbers = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};
 
// 排序
std::ranges::sort(numbers);

// 查找
auto it = std::ranges::find(numbers, 5);
if (it != numbers.end()) {
  std::cout << "Found at index: " << std::distance(numbers.begin(), it) << '\n';
} else {
  std::cout << "Not found\n";
}

// 变换
auto squared_numbers = numbers | std::views::transform([](int num) { return num * num; });

// 筛选
auto even_numbers = numbers | std::views::filter([](int num) { return num % 2 == 0; });

// 访问/切片
// 访问前5个元素
for (auto num : numbers | std::view::take(5)) {
  cout << num << ' ';
}
// 从第三个元素开始访问
for (auto num : numbers | std::view::drop(3)) {
  cout << num << ' ';
}
// 指定访问范围为2到6
for (auto num : numbers | std::view::slice(2, 6)) {
  cout << num << ' ';
}
```

可见，使用ranges中提供的排序和查找算法时，不再需要像algorithm一样指定起始和结束迭代器，而是直接传入原容器即可。此外，关于ranges提供的变换和筛选操作，还需要引入一个新概念std::views，也就是范围视图。它是一种惰性求值的机制，仅在需要时才会计算或生成范围中的元素，而不会立即生成或复制整个范围的元素。

### corountine
