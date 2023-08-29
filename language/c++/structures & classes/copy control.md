# 拷贝控制

- [拷贝控制](#拷贝控制)
  - [拷贝](#拷贝)
    - [直接初始化 vs 拷贝初始化](#直接初始化-vs-拷贝初始化)
    - [拷贝构造函数](#拷贝构造函数)
    - [拷贝赋值运算符](#拷贝赋值运算符)
    - [阻止拷贝](#阻止拷贝)
      - [explicit](#explicit)
      - [= deleted](#-deleted)
      - [private](#private)
  - [移动](#移动)
    - [右值引用](#右值引用)
    - [移动构造函数](#移动构造函数)
    - [移动赋值函数](#移动赋值函数)
  - [析构](#析构)
    - [析构函数](#析构函数)
  - [参考](#参考)

## 拷贝

### 直接初始化 vs 拷贝初始化

**例子**:

``` c++
string dots(10, '.');              // 直接初始化
string s(dots);                    // 直接初始化
string s1 = dots;                  // 拷贝初始化
string s2 = "9999-99-9";           // 拷贝初始化
string s3 = string(10, '.');       // 拷贝初始化
```

当使用直接初始化时，我们实际上是要求编译器使用普通的函数匹配来选择与我们提供的参数最匹配的构造函数。

当我们使用拷贝初始化时，我们要求编译器将右侧运算对象拷贝到正在创建的对象中。

拷贝初始化通常使用拷贝构造函数来完成。但是，如果一个类有一个移动构造函数，那么拷贝初始化有时会使用移动构造函数来完成。

**push vs emplace**：

标准库容器中的`push`成员，会对其元素进行拷贝初始化；

而`emplace`成员，则会对其元素进行直接初始化。

**其他拷贝初始化发生的情况**：

拷贝初始化不仅在使用等号定义变量时发生，对象作为实参传递给非引用类型的形参时也会发生：

### 拷贝构造函数

**拷贝构造函数**：

如果一个构造函数的第一个参数是自身类类型的引用，且任何额外参数都有默认值，则此构造函数是拷贝构造函数。

**合成拷贝构造函数**：

如果我们没有定义一个拷贝构造函数，编译器会为我们定义一个。

与合成默认构造函数不同，即使我们定义了其他构造函数，编译器也会为我们合成一个拷贝构造函数。

### 拷贝赋值运算符

**重载赋值运算符**：

重载运算符本质是函数，其名字由`operator`关键字后续接要定义的运算符号组成。因此，赋值运算符就是一个名为`operator=`的函数。

**合成拷贝赋值运算符**：

与拷贝构造函数一样，如果一个类未定义它自己的拷贝赋值运算符，编译器会为它生成一个合成拷贝赋值运算符。

### 阻止拷贝

#### explicit

我们可以将拷贝构造函数声明为`explicit`来要求必须显式的使用拷贝构造函数。

例如：

``` c++
explicit void func(vector<int> a);

//错误，不能使用一个explicit的拷贝构造函数来拷贝一个实参
func(10);
//正确，使用一个int构造一个临时的vector，再调用拷贝构造函数
func(vector<int>(10));
```

#### = deleted

我们可以通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数来阻止拷贝，即：在函数的参数列表后面加上`=deleted`来指出我们希望将它定义为删除的。

#### private

此外，我们也可以将拷贝构造函数和拷贝赋值运算符声明为`private`的来阻止拷贝。

## 移动

### 右值引用

为了支持移动操作，新标准引入了一种新的引用类型——`右值引用(rvalue reference)`。

所谓的右值引用就是必须绑定到右值的引用。

> 左值和右值是表达式的属性。
> 一般而言，一个左值表达式表示的是一个对象的省份，而一个右值表达式表示的是对象的值。

我们通过 && 而不是 & 来获取右值引用。

右值引用有一个重要的性质，只能绑定到一个将要销毁的对象。

类似任何引用，一个右值引用也不过是某个对象的另一个名字而已。

对于常规引用（为了和右值引用区分开来，我们可以将其称之为`左值引用(lvalue reference)`），我们不能将其绑定到要求转化的表达式、字面常量或是返回右值的表达式。

### 移动构造函数

如果一个构造函数的第一个参数是自身类类型的右值引用，且任何额外参数都有默认值，则则此构造函数是移动构造函数。

除了完成资源移动之外，移动构造函数还必须保证移动后源对象处于销毁状态——它是无害的。特别是，一旦资源移动完成，源对象就必须不再指向被移动资源——这些资源的所有权已经归属到新创建的对象。

### 移动赋值函数

## 析构

### 析构函数

## 参考

- [C++ Primer](file:///D:/ZhY/Workspace/Notes/repick_note/language/c++/detailed/C++%20Primer%E7%AC%AC%E4%BA%94%E7%89%88.pdf)