# 数据模型

- [数据模型](#数据模型)
  - [Aggregate Class](#aggregate-class)
  - [POD](#pod)
  - [Trivial](#trivial)
  - [Standard Layout](#standard-layout)
  - [Class Object Size Calculation](#class-object-size-calculation)
  - [Reference](#reference)

## Aggregate Class

## POD

## Trivial

## Standard Layout

## Class Object Size Calculation

**数据对齐**：

类中成员变量的大小和顺序会影响到整个类对象的大小。编译器会根据各成员变量的数据类型进行内存对齐，以提高访存效率。一般来说，当较大的数据类型放在前面时，较小的数据类型会被填充至较大类型的整数倍。这个过程中可能会产生一些填充字节。比如：

```c++
struct MyStruct {
  char c;     // 1 byte
  int i;      // 4 bytes
  short s;    // 2 bytes
};
```

如果不考虑对齐，这个结构体的大小应该是 1 + 4 + 2 = 7 个字节。但实际上，编译器会进行数据对齐，使得结构体的大小为 12 个字节。具体过程如下:

- char c 占 1 个字节，没有对齐要求，放在起始位置。
- int i 需要 4 字节对齐，所以在 c 之后要空出 3 个字节。
- short s 需要 2 字节对齐，在 i 之后再空出 2 个字节。

因此，整个结构体大小被扩展到 12 个字节，满足各成员变量的对齐要求。这种对齐策略可以提高访存效率，但同时也会浪费一些内存空间。

**有虚函数**：

如果类有虚函数，它会有一个指向虚函数表（vtable）的指针，这个指针的大小需要被计算在内。虚函数不会影响成员的对齐，但会增加类的大小。

**有static成员变量**：

static成员变量不属于类的任何对象，而是属于整个类。因此static成员变量不会计入类对象的大小。它们是存储在类的数据段中的全局变量。

## Reference

[聚合类型与POD类型](https://www.cnblogs.com/jerry-fuyi/p/12854248.html)
