# C++ 11 New Features

> [Youtube - Bo Qian - Modern C++](https://www.youtube.com/watch?v=U6mgsPqV32A&list=PL5jc9xFGsL8FWtnZBeTqZBbniyw0uHyaH)学习笔记

## Initializer List

All the relevant STL containers have been updated to accept initializer_list as parameter to initial.

We can define our own initializer_list constructor to get our class this feature.

``` c++
#include<initializer_list>
template<typename T>
class Vector {
public:
    Vector(const initializer_list<T>& iList) {
        size = iList.size();
        capacity = 2*iList.size();
        arr = new T[capacity];
        for (auto itr = iList.begin(), int idx = 0; itr != iList.end(); itr++, idx++) {
            arr[idx] = *itr;
        }
    }
private:
    T *arr;
    size_t size, capacity;
}
```

## auto type

## foreach

This works on any class that has begin() and end().

``` c++
vector<int> v;
for (auto i : v) {
    cout << i << ' '; 
}
cout << endl;
```

## nullptr

C++ 11 introduces a new key word `nullptr`, which is dedicated to replace `NULL` in C++ 03, because of the ambiguity of `NULL`.

``` c++
void func(int i);
void func(char* p);

int main() {
    func(NULL); // call who? Ambiguity
    func(nullptr); // call func(char* p)
}
```

## enum class

``` c++
enum class apple {green, red};
enum class orange {big, small};

apple a
```

## static_assert

C++ 11 provides a new macro called `static_assert()`, which allows assertion check in compile-time.

``` c++
//run-time assert
assert();
//compile-time assert
static_assert();
```

## delegating constructor

``` c++
//C++ 03
class dog {
public:
    dog() { init(); }
    dog(int a) { init(); doOtherThing(); }
};
/* Cons:
 * 1. Cumbersome code
 * 2. init() could be invoked bt other functions
*/

//C++ 11
class dog {
public:
    dog() { ... }
    dog(int a) : dog() { doOtherThing(); }
};
// Limition : dog() has to be called first. 
```

The constructor execution order for class objects in C++

can be:

+ Constructors of Virtual base classes are executed, in the order that they appear in the base list.
+ Constructors of nonvirtual base classes are executed, in the declaration order.
+ Constructors of class members are executed in the declaration order (regardless of their order in the initialization list).
+ The body of the constructor is executed.

## override

C++ 11 introduces a new key word `override`, which allows to specifically  tell compiler the function is an override function from base class.

## final

C++ 11 introduces a new key word `final`, which allows to prohibit function or class from deriveing.

## Compiler Generated Default Constructor

``` c++
class dog{
public:
    dog() = default;
    dog(int i);
};
```

## delete

``` c++
class dog{
public:
    dog() = default;
    dog(const dog&) = delete;
};
```

## constexpr

``` c++
constexpr int A() { return 3; }
int arr[A() + 3];
```

## lambda function
