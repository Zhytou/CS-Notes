# 类

- [类](#类)
  - [访问控制与封装](#访问控制与封装)
    - [访问说明符](#访问说明符)
    - [友元](#友元)
  - [构造函数](#构造函数)
  - [类的静态成员](#类的静态成员)

## 访问控制与封装

### 访问说明符

- `private`：仅自身和友元可以访问。

- `protected`：仅自身、派生类和友元可以访问。
  - 注意： 派生类成员或者友元只能通过派生类对象来访问基类的受保护成员。

``` c++
class Base{
  protected:
    int n;
};

class Test : public Base{
  // 能访问
  friend void func1(Test&);
  // 不能访问
  friend void func1(Base&);
};
```

- `public`：全部公开。

问题：派生类的友元能够访问基类的`private`成员吗？

答案：不可以（实践检验）。

``` c++
#include <iostream>

using namespace std;

class Base {
 private:
  int n;
};

class Derived : public Base {
 public:
  friend FriendClass;
  friend void printN(Derived& d);
};

class FriendClass {
 public:
  void print(Derived& d) { cout << d.n << endl; };
};

void printN(Derived& d) { cout << d.n << endl; };

int main() {
  Derived d;
  FriendClass fc;
  fc.print(d);
}
```

### 友元

类可以允许友元函数或者友元类访问它的非共有成员。

成为友元只需要在类内增加一条以`friend`关键字开始的声明语句即可。但要注意的是，友元的声明仅仅指定了访问的权限，而非一个通常意义上的声明。因此，如果我们希望使用某个友元类或友元函数，就必须要在友元声明之外专门对友元进行一次声明。

## 构造函数

构造函数必须初始化`const`和引用成员。

最后令构造函数初始化值顺序与成员声明的顺序保持一致。而且如果如果可能的话，尽量避免使用某些成员初始化其他成员。

如果一个构造函数为所有参数都提供了默认实参，则它实际上也定义了默认构造函数。

C++11 新标准扩展了构造函数初始值的功能，使得我们可以定义所谓的委托构造函数。

可以将构造函数声明为`explicit`阻止构造函数定义的隐式转换。

## 类的静态成员

**访问**：

一般来说，我们使用作用域运算符直接访问静态成员。

除此之外，尽管静态成员不属于某个类的对象，但我们仍可以使用类的对象、引用或者指针来访问静态成员。

**声明**：

我们通过在成员的声明之前加上关键字`static`使得其与类关联在一起。和其他成员一样，静态成员可以是`private`的或`public`的。

**定义**：

和其他成员函数一样，我们既可以在类内部也可以在类外部定义静态成员函数。当在类外部定义时，注意不能重复`static`关键字。

此外，我们一般不能在类内部初始化静态成员变量。相反的必须在类的外部定义和初始化每个静态成员。
