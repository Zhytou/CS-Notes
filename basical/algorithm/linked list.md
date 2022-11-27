# 链表

- [链表](#链表)
  - [基础概念](#基础概念)
  - [经典例题](#经典例题)
    - [基础题](#基础题)
    - [中等题](#中等题)
    - [难题](#难题)

## 基础概念

定义：

技巧：

- 快慢指针：寻找倒数第K个节点或中点，判断链表是否有环等
- dummy节点

## 经典例题

### 基础题

[141](https://leetcode.cn/problems/linked-list-cycle/)

- 思路：经典地利用快慢指针判断是否有环。
- 代码：

``` c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        auto slow = head, fast = head;
        while (fast != nullptr && slow != nullptr)
        {
            slow = slow->next;
            fast = fast->next;
            if (fast != nullptr) {
                fast = fast->next;
            } else {
                break;
            }
            if (slow == fast) {
                return true;
            }
        }
        return false;
    }
};
```

### 中等题

[142](https://leetcode.cn/problems/linked-list-cycle-ii/)

- 思路：首先利用快慢指针判断是否有环，无环则直接返回空指针；反之，计算出环长度，将找环入口转换为找出倒数第N个节点。
- 代码：

``` c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        auto slow = head, fast = head;
        int slowi = 0, fasti = 0;
        while (fast != nullptr && slow != nullptr)
        {
            slow = slow->next;
            slowi += 1;
            fast = fast->next;            
            if (fast != nullptr) {
                fast = fast->next;
                fasti += 2;
            } else {
                break;
            }
            if (slow == fast) {
                break;
            }
        }
        if (slow == nullptr || fast == nullptr) {
            return nullptr;
        }
        int k = fasti - slowi;
        slow = head;
        fast = head;
        while (k > 0)
        {
            fast = fast->next;
            k -= 1;
        }
        while (slow != fast)
        {
            slow = slow->next;
            fast = fast->next;
        }
        return slow;
    }
};
```

### 难题
