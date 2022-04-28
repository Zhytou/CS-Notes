# C++ 11 Weak Pointers

> [Youtube - Bo Qian - Modern C++](https://www.youtube.com/watch?v=EWoMjuN5OH4&list=PL5jc9xFGsL8FWtnZBeTqZBbniyw0uHyaH&index=9) 学习笔记

**cyclic references**

``` c++
class Dog{
private:
    shared_ptr<Dog> myFriend;
public:
    string name;
    Dog(string s) : name(s) {};
    void makeFriend(shared_ptr<Dog> other) { myFrined = other; }
}

int main() {
    shared_ptr<Dog> p1("d1"), p2("d2");
    p1->makeFriend(p2);
    p2->makeFriend(p1);
}
```

The code above get a cyclic reference which will cause memory leak.

The solution is to replace the `shared_ptr` in the Dog class with `weak_ptr`.

`weak_ptr` has no ownership of the pointed objects, which is similar to raw pointer.

``` c++
class Dog{
private:
    weak_ptr<Dog> myFriend;
public:
    string name;
    Dog(string s) : name(s) {};
    void makeFriend(shared_ptr<Dog> other) { myFrined = other; }
    
    //std::weak_ptr::lock() creates a shared_ptr over the weak_ptr to ensure the object is not null.
    void showFriend() {
		cout << "My friend is: " << myFriend.lock()->myFriend << endl;
    }
}

int main() {
    shared_ptr<Dog> p1("d1"), p2("d2");
    p1->makeFriend(p2);
    p2->makeFriend(p1);
}
```

