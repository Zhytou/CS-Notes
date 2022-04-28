# C++ 11 User Defined Literals

> [Youtube - Bo Qian - Modern C++](https://www.youtube.com/watch?v=IOkgBrXCtfo&list=PL5jc9xFGsL8FWtnZBeTqZBbniyw0uHyaH&index=3) 学习笔记

Literals are constants. C++ has 4 kinds of literals:

- integer
- floating point
- character
- string



When the units translation are required, the user defined literals can be helpful.

``` c++
//User defined literals
constexpr long double operator"" _cm(long double x) {return x * 10; }
constexpr long double operator"" _m(long double x) {return x * 1000; }
constexpr long double operator"" _mm(long double x) {return x; }

int main() {
    long double height = 3.4_cm;
    cout << (height + 13.0_m) << endl;
    cout << (130.0_mm / 13.0_m) << endl;
}
```

Using key word `constexpr` will make compilers get the result before runtime, which greatly improve the efficiency.

## References

+ [C++ Variables, Literals and Constants](https://www.programiz.com/cpp-programming/variables-literals)

+ [cppreference.com - user defined literals](https://en.cppreference.com/w/cpp/language/user_literal)