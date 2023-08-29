# 结构体 & 联合

- [结构体 \& 联合](#结构体--联合)
  - [结构体](#结构体)
    - [C中的struct](#c中的struct)
    - [C++中的struct](#c中的struct-1)
    - [计算结构体大小](#计算结构体大小)
  - [联合](#联合)
  - [参考](#参考)

## 结构体

### C中的struct

值得注意的是，C语言定义的结构体名字是struct+名称，想要使用这个新类型定义变量，必须加上struct，或使用typedef定义别名。比如：

```c++
struct list_node {
  int val;
  struct list_node* next;
};

struct list_node* head;
```

### C++中的struct

struct类似class，也能定义成员变量和成员函数，但struct默认访问修饰符为public，而class为private。

### 计算结构体大小

**原则**：

- 结构体变量中成员的偏移量必须是该成员大小的整数倍（数字零被认为是任意整数的公倍数）；
- 结构体大小必须是所有成员的整数倍数，也即所有成员大小的公倍数。

**例子**：

``` c++
struct A {
  char a;
  int b;
};

struct B : A {
  int b;
  void func();
};
```

- 成员a自身对齐值为1，存放的起始地址为0x0000；
- 成员b自身对齐值为4，存放的起始地址为0x0004；
- 整个结构体占据8个字节，为所有成员大小整数倍数。

**作用**：

编译器按照4字节对齐，是为了避免多次内存I/O。

## 联合

一个`union`可以拥有多个数据成员，但在任意时刻只有一个数据成员可以有值。

当我们给`union`的某个成员赋值之后，该`union`的其他成员就变成未定义状态了。

分配给一个`union`对象的存储空间至少要能容纳它的最大数据成员。

默认情况下，`union`的成员都是公有的，这一点与`struct`相同。

## 参考

- [结构体字节对齐规则](https://www.cnblogs.com/heart-flying/p/9556401.html)
