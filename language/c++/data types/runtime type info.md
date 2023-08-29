# RTTI

- [RTTI](#rtti)
  - [dynamic\_cast运算符](#dynamic_cast运算符)
  - [typeid运算符](#typeid运算符)

RTTI（Run-Time Type Information，运行时类型信息）是一项功能，提供了在运行时获取和操作对象的类型信息的能力。RTTI 允许程序在运行时确定对象的实际类型，并进行类型检查和类型转换。

在C++中，RTTI的功能主要由两种运算符实现：

- typedid运算符，用于返回表达式类型。
- dynamic_cast运算符，用于将基类的指针或引用安全地转换成派生类的指针或引用。

## dynamic_cast运算符

## typeid运算符

typedid表达式的形式typeid(e)，其中e可以任意表达式或类型的名字。
