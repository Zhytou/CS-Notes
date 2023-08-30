# RTTI

- [RTTI](#rtti)
  - [dynamic\_cast运算符](#dynamic_cast运算符)
  - [typeid运算符](#typeid运算符)

RTTI（Run-Time Type Information，运行时类型信息）是一项功能，提供了在运行时获取和操作对象的类型信息的能力。RTTI 允许程序在运行时确定对象的实际类型，并进行类型检查和类型转换。

在C++中，RTTI的功能主要由两种运算符实现：

- typedid运算符，用于返回表达式类型。
- dynamic_cast运算符，用于将基类的指针或引用安全地转换成派生类的指针或引用。

## dynamic_cast运算符

**功能**：

`dynamic_cast`主要用于将基类指针（或引用）转换为派生类指针（或引用）。

其中，对指针进行`dynamic_cast`，失败返回null，成功返回派生类对象指针；

对引用进行`dynamic_cast`，失败抛出一个异常，成功返回派生类对象引用。

**使用**：

``` c++
dynamic_cast<type*>(expression);
dynamic_cast<type&>(expression);
dynamic_cast<type&&>(expression);
```

由于`dynamic_cast`的功能，所以也要求目标类型`type`和源对象`expression`类型必须满足子父类关系。

## typeid运算符

typedid表达式的形式typeid(e)，其中e可以任意表达式或类型的名字。
