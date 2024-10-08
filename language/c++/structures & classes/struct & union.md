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

**普通函数**：

当一个类中有普通函数时，它并不会计入该对象大小。这很好理解，因为如果每个对象中都有相同的指针，去指向该成员函数地址，显然是一种空间的浪费。

**虚函数**：

不同的是，如果一个类有虚函数，那么它的对象会有一个指向虚函数表（vtable）的指针。因为每个对象可能会指向不同的虚表（运行时多态），所以需要一个虚表指针。正因此，在计算该对象大小时，虚表指针的大小是需要被计算在内的。该指针同样会影响成员的对齐，它在32位机器上占4字节，在64位机器上则有8字节。

**有static成员变量**：

static成员变量不属于类的任何对象，而是属于整个类。因此static成员变量不会计入类对象的大小。它们是存储在类的数据段中的全局变量。

**sizeof运算符**：

sizeof是一个关键字，它是一个编译时运算符，用于判断变量或数据类型的字节大小。值得注意的是，正因为sizeof是一个编译时运算符，它计算c数组和指针有很大不同，比如：

```c++

char a[10];
char *b;

// 输出 10 8（64位机器指针大小） 
cout << sizeof(a) << ' ' << sizeof(b) << endl;
```

此外，无论输入类型还是对象，sizeof总是返回对象大小，比如：

```c++
struct A {
    static int i;
};

int main() {
    A a;
    // output: 1 1
    std::cout << sizeof(A) << ' ' << sizeof(a) << std::endl;
}
```

因为上述类A只有一个static成员变量，所以输出为1。可见使用类或者类对象得出的大小都是对象大小。

## 联合

一个`union`可以拥有多个数据成员，但在任意时刻只有一个数据成员可以有值。当我们给`union`的某个成员赋值之后，该`union`的其他成员就变成未定义状态了。因此，分配给一个`union`对象的存储空间至少要能容纳它的最大数据成员。

默认情况下，`union`的成员都是公有的，这一点与`struct`相同。

## 参考

- [结构体字节对齐规则](https://www.cnblogs.com/heart-flying/p/9556401.html)
