# C++函数式编程

- [C++函数式编程](#c函数式编程)
  - [简介](#简介)
    - [编程范式](#编程范式)
    - [函数式编程](#函数式编程)
  - [可调用对象](#可调用对象)
    - [仿函数类](#仿函数类)
    - [lambda表达式](#lambda表达式)
  - [标准库对函数式编程的支持](#标准库对函数式编程的支持)
    - [可调用对象包装器](#可调用对象包装器)
    - [绑定器](#绑定器)
    - [引用包装器](#引用包装器)

## 简介

### 编程范式

编程范式(Programming Paradigm)是一种编程风格或方法论，用于描述这类程序共有的结构或行为方式。常见的编程范式包括：指令式编程、面向对象编程、声明式编程、元编程和函数式编程等等。

其中，在**指令式编程(Imperative Programming)**风格下，程序由一系列的命令或语句组成，计算机按照这些命令执行操作。这也是最基础的编程范式。而**过程式编程(Procedural Programming)**派生自指令式编程，有时也被视为其同义词，主要特点是以流控制为中心。

**面向对象编程(Object-Oriented Programming)**是以对象为中心的编程范式，将数据和操作数据的方法封装在对象中，通过对象之间的交互来实现程序的功能。面向对象编程强调封装、继承和多态等概念，使得程序更易于理解、扩展和维护。一般来说，OOP的实现可分为以C++和Java为代表的基于类或以JavaScript为代表的基于原型实现。

**声明式编程(Declarative Programming)**是一种描述问题的方式，而不是解决问题的方式。程序员描述需要实现的结果，而不是具体的步骤。常见例子包括SQL和HTML。

**元编程(Meta Programming)**是指在程序运行时操作程序本身的编程技术。以C++为例，它实现元编程的方法包括：

- 宏：预处理宏允许程序员在编译时处理代码；
- 模板：根据SFINAE原则，使用萃取器、concept和if constexpr等方法来生成特定的代码实例；
- constexpr：可用于在编译时计算表达式的值。

### 函数式编程

函数式编程(Functional Programming)是一种编程范式，通过函数的求值和组合来构造程序。它强调使用纯函数(Pure Function)，即没有状态和副作用、返回值只依赖于参数的函数。比如：

```c++
vector<int> v1(10);
// 对v1进行自增填充
itoa(v1.begin(), v1.end(), 1);
auto v2 = v1 | views::filter([](int x) { return x % 2 == 0; }) | views::transform([](int x) { return x * x; });
int sum = accumulate(v2.begin(), v2.end(), 0);
```

由此可见，函数式编程的优势包括：

- 易于测试和维护：函数式编程鼓励使用纯函数和不可变数据结构，这使得代码更容易测试和维护。当你查看一个纯函数时，你不需要考虑外部状态，只需要关注函数的输入和输出。
- 代码简洁和可读性：函数式编程通常会产生更简洁、更具表达力的代码。这是因为函数式编程鼓励你使用函数组合（Function Composition）和高阶函数（Higher-Order Functions）来解决问题。
- 并发和多线程：在多核处理器和多线程环境中，函数式编程因其不可变性和无状态性而变得尤为重要。这些特性使得函数式编程在并发编程中非常有用。

## 可调用对象

在C++中，可调用对象就是能够对其使用调用运算符的对象或表达式，其大致包括：

- 函数指针；
- 具有operator()成员函数的类对象（仿函数）；
- 一个可被转为函数指针的类对象；
- 是一个类成员函数的指针。

### 仿函数类

仿函数（Functor）就是可以被视为函数或函数指针的对象。一个定义了operator()成员函数的类就能够被称为仿函数类。因为它重载了函数调用运算符，所以我们可以像使用函数一样使用该类对象，故而被称作仿函数。其最大的好处是，这样的类对象能够存储状态，所以比普通函数更加灵活。

### lambda表达式

> A lambda expression creates an nameless functor, it's syntactic sugar.

在C++中，lambda表达式允许在代码中定义匿名函数，可以看作是一种简化的仿函数写法。换句话说，定义一个lambda表达式也就创建了一个未命名的仿函数对象。例如：

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

**表达式结构**：

一般来说，一个lambda表达式具有如下形式`[capture list](parameter list) -> return type {function body}`。

**捕获列表**：

其中，capture list是一个lambda表达式所在作用域中定义的局部变量列表，通常为空。lambda表达式使用捕获列表来明确它将使用的外部变量，可按能否修改分为值捕获和引用捕获的方式，具体使用方法如下：

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

**值可修改**：

如果我们希望改变一个被捕获的变量的值，就必须在参数列表加上关键字`mutable`。例如：

```c++
int sz;

// 值捕获sz
auto cmp = [sz](const string& s) mutable {
  return s.size() >= sz;
}
```

事实上，值捕获，或者说默认状态下的lambda表达式就有点类似使用const修饰的类成员函数，所以必须使用mutable修改传入的值。

**闭包与lambda表达式**：

闭包(Closure)是函数式编程的一个重要概念。它指的是一个函数以及从该函数作用域内引用了自由变量的环境组合在一起的实体。

在C++中，lambda表达式就可以实现闭包:lambda表达式可以直接访问并使用它外层函数作用域内的变量，这些变量不在lambda函数内声明。而lambda本身又可以作为一个独立的可调用实体传递给其他函数。

**lambda捕获unique_ptr**：

早期C++的lambda表达式只有值捕获、引用捕获和隐式捕获三种方式，因此其难以捕获unique_ptr这种movable的对象。为了改善这个问题，C++14引入了广义捕获的特性(Generalized Lambda Captures)。这种特性允许lambda表达式在捕获列表中定义任意的本地变量来帮助你捕获，比如：

```c++
// a unique_ptr is move-only
auto u = make_unique<some_type>(some, parameters); 

// move the unique_ptr into the lambda
go.run([u = move(u)]{do_something_with(u);});
```

**递归lambda**：

[How to make a recursive lambda](https://stackoverflow.com/questions/2067988/how-to-make-a-recursive-lambda)

## 标准库对函数式编程的支持

### 可调用对象包装器

std::function是一个可变参类模板，是一个通用的函数包装器（Polymorphic Function Wrapper）。std::function的实例可以存储、复制和调用任何可复制构造的可调用目标，包括普通函数、成员函数、类对象（重载了operator()的类的对象）、lambda表达式等。是对C++现有的可调用实体的一种类型安全的包裹（相比而言，函数指针这种可调用实体，是类型不安全的）。

std::function中存储的可调用对象被称之为std::function的目标。若std::function中不含目标，调用不含目标的std::function会抛出std::bad_function_call 异常。

和其他对象一样，可调用的对象也有类型。例如：每个lambda表达式有它自己唯一的类类型。然而两个不同类型的可调用对象可能共享一种调用形式(Call Signature)。

std::function是一个类模板，一个特定类型T的std::function可以存储拥有这种调用形式的可调用对象。例如：

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

### 绑定器

std::bind用来将可调用对象与其参数一起进行绑定。绑定后的结果可以使用std::function进行保存，并延迟调用到任何我们需要的时候。准确来说，它包含两大作用：

- 将可调用对象与其参数一起绑定成一个仿函数对象；
- 将多元（参数个数为n）可调用对象转成一元或（n-1）元可调用对象，即只绑定部分参数。

其具体使用方法如下：

```c++
int add(int x, int y) {
  return x + y;
}

auto f = std::bind(add, 1, 2);
int result = f(); // 调用加法函数并传入已绑定参数
```

### 引用包装器

std::reference_wrapper是标准库提供的一种模板类，它能够将一个引用对象包装成一个可复制和赋值的对象。其核心作用就是在函数式编程或多线程编程时传递引用参数。和它密切相关的是另外两个标准库函数：std::ref和std::cref。二者分别用于返回普通和const的std::reference_wrapper对象。比如：

```c++
void f(int& n1, int& n2, const int& n3)
{
  std::cout << "In function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
  ++n1; // increments the copy of n1 stored in the function object
  ++n2; // increments the main()'s n2
  // ++n3; // compile error
}

int main() {
  int n1 = 1, n2 = 2, n3 = 3;0
  std::function<void()> bound_f = std::bind(f, n1, std::ref(n2), std::cref(n3));
  f();

  std::thread th(f, n1, std::ref(n2), std::cref(n3));
  th.join();
}
```
