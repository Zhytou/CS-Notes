# C++ 11 Shared Pointers

> [Youtube - Bo Qian - Modern C++](https://www.youtube.com/watch?v=IOkgBrXCtfo&list=PL5jc9xFGsL8FWtnZBeTqZBbniyw0uHyaH&index=3) 学习笔记

``` c++
int main() {
    
    //An object should be assigned to a smart pointer as soon as it is created.Raw pointer should not be used.
    
    //Bad idea
    Dog* d = new Dog("Tank"); 
    shared_ptr<Dog> p(d); //p.use_count() = 1;
    shared_ptr<Dog> p2(d);//p.use_count() = 1;
    
    //OK but not perfect
    shared_ptr<Dog> = p(new Dog("Tank"));
    
    //Perfect, faster and exciption safe
    shared_ptr<Dog> = make_shared<Dog>("Tank")
}
```

Though the third initialization `make_shared` is perfect, sometimes we have to use the second way like the case that we want to use custom deleter instead of default deleter.

``` c++
int main() {
    shared_ptr<Dog> p(new Dog[3]);//dog[1] and dog[2] have memory leak.
    shared_ptr<Dog> p2(new Dog[3], [](Dog* p) {
        delete [] p;
    });// no memory leak
    
    shared_ptr<Dog> p3 = make_shared<Dog>("Tank");
    Dog* d = p3.get()//returns the raw pointer.
}
```

However, With C++ 17, `shared_ptr` can be used to manage a dynamically allocated array. The `shared_ptr` template argument in this case must be `T[N]` or `T[]`.

``` c++
shared_ptr<int[]> sp(new int[10]);
```
