# 元编程

- [元编程](#元编程)
  - [模板起源](#模板起源)
    - [宏与泛型](#宏与泛型)
    - [零成本抽象](#零成本抽象)

**元编程定义**：

元编程指的是编写那些能够操作或生成代码的代码。一些常见的元编程表现形式包括：

- 反射：程序能够在运行时获取自身以及其他程序信息；
- 宏：程序经过预处理操作后会自动生成或修改代码；
- 面向切面编程：通过插件机制将代码分割成不同的关注点，在编译或运行期间动态组合代码模块。

**C++元编程**：

在C++中，元编程的流派可以分为：

- Constexpr all the things!
- 模板元编程
- 两者结合

其中，模板编程指的使用C++模板来在编译期生成对应类型的代码从而复用一份代码。而constexpr技术则更偏向编译期计算

**泛型编程**：

泛型编程和元编程从概念上并无直接联系，它指的是编写可复用逻辑的代码，这些代码可以适用于多种类型的数据。但C++的模板编程就可以实现泛型编程。

## 模板起源

模板起初是为了支持泛型替代而设计的语法。它的原理和宏类似，在编译器根据实际调用时传入的类型参数替换占位符，生成专属该类型的代码。但和宏相比，它具有类型安全、编译期检查错误以及零成本抽象的特点。

### 宏与泛型

在模板语法还未出现之时，依靠宏来实现泛型容器的方法如下：

```c++
// collection.h
struct Collection_ ## TYPE {
    TYPE *array; 
    size_t size, n;
};

#ifdef INSTANCE 
Collection_ ## TYPE make_Collection_ ## TYPE(size_t sz) {
    // ...
}
#endif

// main.cpp
#define INSTANCE

#define TYPE int 
#include "collection.h"
#undef TYPE

#define TYPE double
#include "collection.h"
#undef TYPE

int main() {
    Collection_int lstInt = make_Collection_int(5); 
    Collection_double lstDouble = make_Collection_double(5);
}
```

利用宏，实现了对collection.h中代码的复用。

### 零成本抽象

> C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.
