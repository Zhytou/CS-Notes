# C++ 11 Unique Pointers

> [Youtube - Bo Qian - Modern C++](https://www.youtube.com/watch?v=YeI6V2O5BFE&list=PL5jc9xFGsL8FWtnZBeTqZBbniyw0uHyaH&index=10)

Unique pointer is light weight smart pointer which has exclusive ownership of the pointed object.

``` c++
void func(unique_ptr<Dog> p) {
    //...
}
int main() {
    unique<Dog> p(new Dog);
    p.release();
    // p = nullptr but do not destroy the object
    
    p.reset();
    // p = nullptr and destroy the object
    
    unique_ptr<Dog> p1(new Dog("Gunna"));
    unique_ptr<Dog> p2(new Dog("Somkey"));
    
    p1 = move(p2);
    // Gunna is destroyed
    // p2 = nullptr
    // p1 points to Smokey

    func(move(p1));
    //p1 = nullptr
    //parameter p points to Smokey
    //when out of the func scope, Smokey is destroyed
    
    shared_ptr<Dog> p3(new Dog[3], [](Dog *p) {
        delete [] p;
    });
    // C++ 17
    //shared_ptr<Dog[]> p3(new Dog[3]);
    unique_ptr<Dog[]> p4(new Dog[3]);
}
```
