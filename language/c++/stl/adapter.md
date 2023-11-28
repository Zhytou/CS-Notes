# 适配器

- [适配器](#适配器)
  - [容器适配器](#容器适配器)
  - [函数适配器](#函数适配器)
  - [迭代器适配器](#迭代器适配器)

STL适配器是对现有STL容器进行再包装，以提供新的函数接口和行为。

简单来说,适配器通过包装现有容器:

- 增强/修改容器的接口和功能，比如queue通过deque实现先进先出行为。
- 提供简单易用的接口隐藏底层容器细节，比如stack用易懂接口封装deque。
- 统一不同容器的接口，如vector和deque通过同样的接口操作pair/tuple元素。

## 容器适配器

容器适配器包括stack，queue，priorty-queue。在默认情况下，stack和queue是基于deque实现的，而priorty-queue则是在vector上实现的，并且可以根据第二个实参指定容器的类型，但是一定要符合标准。比如，queue要求有pop_front的操作，因此不能建立在vector的基础上，priorty-queue要求有随机访问的功能，因此建立在vector上。

## 函数适配器

STL中提供了一系列函数适配器，这些适配操作包括连结（Bind）、否定（Negate）、组合（Compose）以及对一般函数或成员函数的修饰。

函数适配器可以封装、修饰一个函数，变成符合我们需要的接口。例如，functional库里提供的适配器bind2nd，可以指定某个仿函数的第二个参数。例如，我们想要获得一个能够判断一个数是否小于10的仿函数，`bind2nd<less<T>, 10>`便能满足需求。

## 迭代器适配器

STL还为迭代器提供了一系列适配器，包括：insert_iterator, reverse_iterator, istream_iterator等，定义在头文件iterator中。通过修饰普通迭代器的接口，使其成为另一种迭代器，发挥不同的作用。
