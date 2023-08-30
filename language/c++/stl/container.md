# 容器

- [容器](#容器)
  - [顺序容器](#顺序容器)
  - [关联容器](#关联容器)
    - [红黑树](#红黑树)
    - [散列表](#散列表)
      - [冲突解决](#冲突解决)
      - [桶与节点](#桶与节点)
  - [实战](#实战)
    - [vector](#vector)
    - [priority\_queue](#priority_queue)
    - [hashtable](#hashtable)
    - [for more STL container implementation](#for-more-stl-container-implementation)
  - [参考](#参考)

## 顺序容器

- vector
- deque
- list

## 关联容器

在C++中，两个主要的`关联容器(associative-container)`类型是`map`和`set`。

- `map`中的元素是一些`关键字—值(key-value)对`；关键字起到索引的作用，值则表示与索引相关联的数据
- `set`中每个元素只包含一个关键字；`set`支持高效的关键字查询操作——检查给定关键字是否存在`set`中

C++ 标准库提供了8个关联容器，这八个容器的不同体现在三个维度上：

- 或者是一个`map`，或者是一个`set`
- 或者要求不重复的关键字，或者允许不重复的关键字
- 按顺序保存元素，或无序保存

### 红黑树

红黑树不仅是一个平衡二叉搜索树，而且还必须满足下列规则：

- 每个节点不是黑色，就是红色；
- 根节点为黑色；
- 如果节点为红，则其子节点必须为黑；
- 任一节点至树尾端的任何路径，所含黑色节点数量必须相同。

### 散列表

#### 冲突解决

**开放定址**：

使用某种探测算法在散列表中寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到。

就是即使key产生冲突，也不会形成链表，而是将所有元素都存入哈希表里。发生冲突时，就以当前地址为基准，进行再寻址的方法去寻址下一个地址，直到找到一个为空的地址为止。

但这种方法存在一个问题，就是删除元素时，只能懒惰删除，即将该位置中元素标记为已删除。因为每个元素不仅关系到自己，还影响查找其他元素。

具体来说，寻找新地址的方法分为三种，分别是线性探查法、平方探查法和双散列函数探查法：

- 线性探测：顺序查找下一个位置，直到找到一个空位置（固定步长1探测）。
- 二次探测：在表的左右位置进行按一定步长跳跃式探测（固定步长n探测）。
- 伪随机探测：根据公式生成一个随机数，作为此次探测空位置的步长（随机步长n探测）。

**链地址法(开链)**：

链接地址法的思路是将哈希值相同的元素构成一个同义词的单链表，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。

也可以进一步优化，将链表变成搜索树，缩短查找元素的时间，如同JAVA中哈希表的做法一般。

**再哈希法**：

就是同时构造多个不同的哈希函数。

这种方法不易产生聚集，但是增加了计算时间。

**建立公共溢出区**：

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

#### 桶与节点

按链地址法定义的散列表中，数组中元素就是桶节点，也就是链表头节点。而链表中元素则是实际存储信息的节点。

## 实战

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

### hashtable

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

### for more STL container implementation

check my repo [SimpleSTL](https://github.com/zhytou/simplestl)

## 参考

- [实现priority_queue](https://blog.csdn.net/loreal8/article/details/106815948)

- [Java集合常见知识总结](https://javaguide.cn/java/collection/java-collection-questions-01.html#java-%E9%9B%86%E5%90%88%E6%A6%82%E8%A7%88)

- [解及哈希冲突的常用方法](https://cloud.tencent.com/developer/article/1672781)
