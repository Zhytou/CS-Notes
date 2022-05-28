# 杂项

## Lambda expression

A lambda expression creates an nameless functor, it's syntactic sugar.

## Functor class

Functors are objects that can be treated as though they are a function or function pointer.

A functor is pretty much just a class which defines the operator(). That lets you create objects which "look like" a function

One nice thing about functors is that unlike regular functions, they can contain state which means that it can be customized.

## 运行时类型识别 RTTI

### typeid

**功能**

查询类型的信息。



### dynamic_cast

**功能**

`dynamic_cast`主要用于将基类指针（或引用）转换为派生类指针（或引用）。

其中，对指针进行`dynamic_cast`，失败返回null，成功返回派生类对象指针；

对引用进行`dynamic_cast`，失败抛出一个异常，成功返回派生类对象引用。

**使用**

``` c++
dynamic_cast<type*>(expression);
dynamic_cast<type&>(expression);
dynamic_cast<type&&>(expression);
```

由于`dynamic_cast`的功能，所以也要求目标类型`type`和源对象`expression`类型必须满足子父类关系。

## PIMPL 具体实现的指针

Pimpl（Pointer to Implementation）是一项实现隐藏、降低耦合性和分离接口的C++技术。

它通过一个私有的成员指针，将指针所指向的类的内部实现数据进行隐藏。

## CRTP 奇异递归模板模式

CRTP（curiously recurring template pattern）是C++模板编程时的一种惯用法：把派生类作为基类的模板参数。

## 参考

+ [C++ Primer]()

**lambda**

+ [stackoverflow - lambda vs functor](https://stackoverflow.com/questions/4686507/lambda-expression-vs-functor-in-c)
+ [all about lambda function in c++](http://www.vishalchovatiya.com/learn-lambda-function-in-cpp-with-example/)

**functor**

+ [stackoverflow - functor](https://stackoverflow.com/questions/356950/what-are-c-functors-and-their-uses)

**RTTI**

+ [C++ Primer 19.2运行时类型识别]()

**PIMPL**

+ [Pimpl技术——编译期封装](https://www.cnblogs.com/KillerAery/p/9539705.html)