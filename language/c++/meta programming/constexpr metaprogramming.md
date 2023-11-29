# constexpr 元编程

**constexpr发展**：

- C++11 引入constexpr简单函数
  - 只允许一条return语句
  - 递归解决问题！简单的数学函数、字符串hash函数
- C++14 放开constexpr约束, 模板变量
  - 泛化constexpr
  - 一些库出现
- C++17 if constexpr、constexpr lambda、折叠表达式
  - 表达力提升
- C++20 constexpr容器、constexpr new、constexpr析构函数、constexpr虚函数、consteval/constinit、lambda模板参数
- constexpr STL algorithms

**constexpr vs 模板**：

- constexpr作为新势力，生态待完善，参考资料少；而模板则体系成熟，拥有大量库；
- constexpr可读性强，维护性强；而模板可读性差，维护性差；
- constexpr编译速度快，跨平台支持强；而模板编译速度慢，跨平台支持弱。

## 深入constexpr

### constexpr常量

**constexpr vs const**：

和const要求结果不可修改不同，constexpr变量必须能在编译期求值得到结果，而constexpr函数则要求函数体内所有计算必须能在编译期完成求值。

换句话说，const只保证引用期不变，但不能保证编译期求值，而constexpr则可以在编译期计算和优化，获得更高效能。

**constexpr lambda**：

C++17起，lambda默认为constexpr，比如：

```c++
// constexpr int fibonacci(int n);
auto fibonacci = [](int n) {
    int a = 0, b = 1;
    for (int c = 0; c < n; ++ c) {
        int t = a + b;
        a = b;
        b = t;
    }
    return a;
};

constexpr auto v = fibonacci(10);
static_assert(v == 55)
```

### constexpr模板常量

**常量别名**：

```c++
template<class T>
constexpr bool is_class_v = std::is_class<T>::value;
```

**表达式计算**：

```c++
template<char c>
constexpr bool is_digit = (c >= '0' && c <= '9');
template<char c>
constexpr bool is_digit_or_dot = (is_digit<c> || c == '.');

static_assert(! is_digit<'x'>);
static_assert(is_digit<'0'>);
```

**模板特化**：

```c++
template<size_t N>
constexpr size_t fibonacci = fibonacci<N - 1> + fibonacci<N - 2>;
template<>
constexpr size_t fibonacci<0> = 0;
template<>
constexpr size_t fibonacci<1> = 1;

static_assert(fibonacci<10> == 55);
```

### if constexpr

if constexpr能够在编译期对constexpr表达式进行条件判断。具体来说，在编译期间，编译器会发现有一个分支确定不会被执行（相当于 if(false) {}），那么这个分支就不会进行编译，直接剔除。

这个特性能够帮助我们对代码进行优化甚至是进行修改。比如，某个平台下完全不支持某个功能，那么这个功能就不应该被编译，过去我们可能依靠宏或者模板的SFINAE特性来帮助我们实现。

```c++
// 依靠宏
#ifdef XXX
  // ...
#else
  // ...
#endif

// 依靠模板SFINAE
// 仅当调用cond为真时，实例化该模板
template<T>
typename enable_if<cond, bool>::type func(){
  // ...
}
```

在有了if constexpr语法之后，我们可以配合着类型萃取或约束（C++20特性）来达成同样的效果，比如：

```c++
template<typename T, typename Tag>
void process(T t, Tag tag) {
  if constexpr(std::is_same_v<Tag, int>) {
    // 处理int标签版本
  } else {
    // 其他标签版本 
  }
}
```

通过这种方法实现跨平台支持，也被称为tag dispatch。

## 参考

- [Constexpr 从11到20](https://netcan.github.io/presentation/constexpr_from_11_20/#/)
- [CppCon Constexpr all the things!](https://github.com/CppCon/CppCon2017/blob/master/Presentations/constexpr%20ALL%20the%20things/constexpr%20ALL%20the%20things%20-%20Jason%20Turner%20and%20Ben%20Deane%20-%20CppCon%202017.pdf)
