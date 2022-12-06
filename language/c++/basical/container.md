# 容器

- [容器](#容器)
  - [顺序容器](#顺序容器)
    - [vector](#vector)
    - [priority\_queue](#priority_queue)
  - [关联容器](#关联容器)
    - [底层原理](#底层原理)
      - [实战](#实战)
    - [冲突解决](#冲突解决)
      - [开放定址](#开放定址)
      - [链地址法](#链地址法)
      - [再哈希法](#再哈希法)
      - [建立公共溢出区](#建立公共溢出区)
  - [参考](#参考)

## 顺序容器

### vector

``` c++
template <typename T>
class Vector {
public:
    Vector() {
        size = 0;
        capacity = 0;
        p = nullptr;
    }
    Vector(size_t _size, const T& _val) {
        size = _size;
        capacity = size << 1;
        p = new T[capacity];
        for (int i = 0; i < size; i++)
            p[i] = _val;
    }
    ~Vector {
        delete [] p;
    }
    void push_back(T val) {
        if (size + 1 > capacity) {
            capacity = capacity << 1;
            T *np = new T[capacity];
            for (int i = 0; i < size; i++)
                np[i] = p[i];
            delete [] p;
            p = np;
        }
        p[size] = val;
        size += 1;
    }
    
    T& operator [](const int& index) {
        assert(index < size);
        return p[index];
    }
private:
    T *p;
    size_t size, capacity;
};
```

### priority_queue

``` c++
//Compare类是Functor,即重载了operator()的类
template <class T, class Container = vector<T>, class Compare = less<T>>
class PriorityQueue {
public:
    //插入数据
    void push(T val) {
        con.push_back(val);
        shiftUp();
    }
    //删除数据
    void pop() {
        swap(con[0], con[con.size() - 1]);
        con.pop_back();
        shiftDown();
    }
    //访问堆顶数据
    const T& top() const {
        return con[0];
    }
    //获取堆大小
    size_t size() {
        return con.size();
    }
    //获取堆是否为空
    bool empty() {
        return con.empty();
    }
private:
    //将堆尾数据向上调整
    void shiftUp() {
        size_t son = con.size() - 1, dad;
        while (son > 0) {
            dad = (son - 1)/2;
            if (cmp(con[dad], con[son])) {
                swap(con[dad], con[son]);
                son = dad;
            }
            else 
                break;
        }
    }
    //将堆首数据向下调整
    void shiftDown() {
        size_t dad = 0, son;
        while (son < con.size()) {
            son = 2*dad + 1;
            if (son + 1 < con.size() && con[son] < con[son + 1])
                son += 1;
            if (cmp(con[dad], con[son]))
                break;
            else {
                swap(con[dad], con[son]);
                dad = son;
            }
        }
    }
    Container con;
    Compare cmp;
}
```

**补充——堆排序**：

算法：

- 首先，自底向上建立堆；
- 接着，将堆顶最大值与堆尾交换，并把前n - 1个数重新堆化。
  - 重复上述步骤，直到堆长度变为0，此时完成排序。

代码：

``` c++
//此处堆化函数必须要有heapSize参数是因为排序时会把堆首与堆尾交换改变堆长度，因此堆化函数必须传入堆长度
void heapify(vector<int>& nums, int start, int heapSize) {
    //当用数组表示树时，若父节点序号为idx，则两个子节点分别为2*idx+1 和2*idx+2
    int dad = start, son = 2*dad + 1;
    while (son < heapSize) {
        //选出两个子节点中较大的
        if (son + 1 < heapSize && nums[son] < nums[son + 1])
            son += 1;
        //若此时父节点已经大于子节点，则已经符合堆规制；否则继续堆化。
        if (nums[dad] < nums[son]) {
            swap(nums[dad], nums[son]);
            dad = son;
            son = 2*dad + 1;
        }
        else
            break;
    }
}

void heapSort(vector<int>& nums) {
    //自下而上的建立堆
    for (int i = nums.size() / 2; i >= 0; i--)
        heapify(nums, cmp, i, nums.size());
    
    
    for (int i = nums.size() - 1; i > 0; i--) {
        //将堆顶最大值与堆尾交换，重新堆化
        //此处必须要传入heapSize参数，因为堆长度减少了1
        swap(nums[0], nums[i]);
        heapify(nums, cmp, 0, i);
    }
    
    return ;
}
```

## 关联容器

在C++中，两个主要的`关联容器(associative-container)`类型是`map`和`set`。

- `map`中的元素是一些`关键字—值(key-value)对`；关键字起到索引的作用，值则表示与索引相关联的数据
- `set`中每个元素只包含一个关键字；`set`支持高效的关键字查询操作——检查给定关键字是否存在`set`中

C++ 标准库提供了8个关联容器，这八个容器的不同体现在三个维度上：

- 或者是一个`map`，或者是一个`set`
- 或者要求不重复的关键字，或者允许不重复的关键字
- 按顺序保存元素，或无序保存

### 底层原理

> 此处，主要是按照Java来梳理的，C++中map和unordered_map能分别对应Java中TreeMap和HashMap

![img](https://javaguide.cn/assets/java-collection-hierarchy.1727461b.png)

- `HashMap`： JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
- `LinkedHashMap`： `LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》open in new window](https://www.imooc.com/article/22931)
- `Hashtable`： 数组+链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的
- `TreeMap`： 红黑树（自平衡的排序二叉树）

#### 实战

[LRU实现](https://leetcode-cn.com/problems/lru-cache/)

``` c++
struct DLinkedNode {
    int key, value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode(): key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int _key, int _value): key(_key), value(_value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    unordered_map<int, DLinkedNode*> cache;
    DLinkedNode* head;
    DLinkedNode* tail;
    int size;
    int capacity;

public:
    LRUCache(int _capacity): capacity(_capacity), size(0) {
        // 使用伪头部和伪尾部节点
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        }
        // 如果 key 存在，先通过哈希表定位，再移到头部
        DLinkedNode* node = cache[key];
        moveToHead(node);
        return node->value;
    }
    
    void put(int key, int value) {
        if (!cache.count(key)) {
            // 如果 key 不存在，创建一个新的节点
            DLinkedNode* node = new DLinkedNode(key, value);
            // 添加进哈希表
            cache[key] = node;
            // 添加至双向链表的头部
            addToHead(node);
            ++size;
            if (size > capacity) {
                // 如果超出容量，删除双向链表的尾部节点
                DLinkedNode* removed = removeTail();
                // 删除哈希表中对应的项
                cache.erase(removed->key);
                // 防止内存泄漏
                delete removed;
                --size;
            }
        }
        else {
            // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
            DLinkedNode* node = cache[key];
            node->value = value;
            moveToHead(node);
        }
    }

    void addToHead(DLinkedNode* node) {
        node->prev = head;
        node->next = head->next;
        head->next->prev = node;
        head->next = node;
    }
    
    void removeNode(DLinkedNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void moveToHead(DLinkedNode* node) {
        removeNode(node);
        addToHead(node);
    }

    DLinkedNode* removeTail() {
        DLinkedNode* node = tail->prev;
        removeNode(node);
        return node;
    }
};
```

### 冲突解决

#### 开放定址

从发生冲突的那个单元起，按照一定的次序，从哈希表中找到一个空闲的单元。然后把发生冲突的元素存入到该单元的一种方法。

开放定址法需要的表长度要大于等于所需要存放的元素。

具体来说，开放定址法又分为：线性探查法、平方探查法和双散列函数探查法。

#### 链地址法

链接地址法的思路是将哈希值相同的元素构成一个同义词的单链表，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。

#### 再哈希法

就是同时构造多个不同的哈希函数。

这种方法不易产生聚集，但是增加了计算时间。

#### 建立公共溢出区

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

## 参考

- [实现priority_queue](https://blog.csdn.net/loreal8/article/details/106815948)

- [Java集合常见知识总结](https://javaguide.cn/java/collection/java-collection-questions-01.html#java-%E9%9B%86%E5%90%88%E6%A6%82%E8%A7%88)

- [解及哈希冲突的常用方法](https://cloud.tencent.com/developer/article/1672781)
