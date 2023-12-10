# 数据成员初始化

在C++11之前，如果有类成员，则只能通过构造函数中的初始化列表使用默认值对其进行初始化。

```c++
// pre C++11 class:
struct SimpleType {
    int field;
    std::string name;
    
    SimpleType() : field(0), name("Hello World") { }
}
```

从C++11开始，语法得到了改进，我们可以在声明的地方初始化字段和名称，比如：

```c++
// since C++11:
struct SimpleType {
    int field = 0;                        // works now!
    std::string name { "Hello World "} // alternate way with { }
    
    SimpleType() { }
}
```

这个特性被称为非静态数据成员初始化（Non-Static Data Member Initialization），简称NSDMI。一般来说，它有这些类型：

```c++
struct S {
    int zero {};       // fine, value initialization
    int a = 10;        // fine, copy initialization    
    double b { 10.5 }; // fine, direct list initialization
    // short c ( 100 );   // err, direct initialization with parens
    int d { zero + a }; // dependency, risky, but fine
    // double e { *mem * 2.0 }; // undefined!
    int* mem = new int(d);
    long arr[4] = { 0, 1, 2, 3 };
    std::array<int, 4> moreNumbers { 10, 20, 30, 40};
    // long arr2[] = { 1, 2 }; // cannot deduce
    // auto f = 1;     // err, type deduction doesn't work
    double g { compute() };

    ~S() { delete mem; }
    double compute() { return a*b; }
};
```

C++14后，聚合类也开始支持持NSDMI，比如：

```c++
truct Point { float x = 0.0f; float y = 0.0f; };

// won't compile in C++11
Point myPt { 10.0f, 11.0f };
// fixed in C++14
Point myPt { 10.0f, 11.0f };
```

到此为止，我们都只讨论了非静态成员变量的初始化，那静态变量呢？

在C++11/14，我们必须在源文件定义静态变量，因为头文件只是声明，比如：

```c++
// xxx.h
struct XXX{
    static int classCtn;
}


// xxx.cc
int XXX::classCtn = 0;
```

在C++17，我们可以给静态变量加上inline的声明，来使其可以在头文件中被定义和初始化，比如：

```c++
// xxx.h C++17
struct XXX{
    static inline int classCtn = 0;
}
```

值得注意的是，这个特性在C++17是仅限于静态成员常量。换句话说C++17将这个特性扩展到了所有静态成员变量。

## 参考

[(Non) Static Data Members Initialization, from C++11 till C++20](https://www.cppstories.com/2015/02/non-static-data-members-initialization/)
