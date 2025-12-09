# **环形链表 II (Linked List Cycle II)** 🔁
## 题目链接 🌐  
[链表中的环 - LeetCode](https://leetcode.cn/problems/linked-list-cycle-lcci/)

---

## 解题思路 🧠

### 思路分析 🧩

这道题目要求我们检测链表中是否存在环，并返回环的起始节点。如果存在环，返回环的起始节点；否则，返回 `NULL`。

### 快慢指针法 (Floyd's Tortoise and Hare Algorithm) 🐢🐇

为了检测链表是否有环，并找到环的起点，我们可以使用 **快慢指针法**，也叫 **Floyd 判圈算法**。这种方法的关键在于：如果链表有环，快慢指针最终会相遇。

### 步骤 📋：

1. **初始化指针** 🧾：
   - 创建两个指针 `slow` 和 `fast`，初始化时都指向链表的头节点。
   - 另外，我们使用一个虚拟节点 `dummy` 来简化边界处理。

2. **判断环的存在** 🔍：
   - 使用快慢指针遍历链表，慢指针每次走一步，快指针每次走两步。
   - 如果链表中没有环，快指针会先到达 `NULL`，此时返回 `NULL`。
   - 如果链表中有环，快慢指针会在环内相遇。

3. **找环的起点** 📍：
   - 当快慢指针相遇时，我们让 `slow` 指针重新指向链表头节点，并保持 `fast` 指针在相遇节点处。
   - 然后，两个指针一起移动，每次走一步，直到它们相遇，那个位置就是环的起始节点。

4. **返回环的起点** 🏁：
   - 返回相遇时的节点，即环的起始节点。

---

### 时间复杂度 ⏱️：

- **O(n)**，其中 `n` 是链表的长度。每个指针最多遍历链表一次。
- 空间复杂度为 **O(1)**，只使用了常数的空间。

---

## 代码实现 💻

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* dummy = new ListNode();
        dummy->next = head;
        ListNode* slow = dummy, *fast = dummy;

        // 快慢指针判断是否存在环
        do {
            if (!fast->next || !fast->next->next)  // 没有环
                return NULL;
            slow = slow->next;
            fast = fast->next->next;

        } while (slow != fast);  // 快慢指针相遇

        slow = dummy;
        // 找到环的起点
        while (slow != fast) {
            slow = slow->next;
            fast = fast->next;
        }
        return fast;  // 返回环的起点
    }
};
```

---

## 关键点 🔑：

- **快慢指针**：快慢指针是解决环问题的一种经典方法，能够高效判断链表是否有环。
- **环的起点**：通过让慢指针重新指向头节点，并与快指针同步移动，找到环的起始位置。
- **虚拟头节点**：虚拟头节点的使用简化了边界情况的处理。

---

## 边界情况 🚨：

1. **链表为空** 🛑：如果链表为空，直接返回 `NULL`。
2. **没有环** ❌：如果链表没有环，返回 `NULL`。
3. **链表存在环** 🔄：如果链表存在环，返回环的起始节点。

---

## 总结 📚

本题通过 **快慢指针** 来判断链表是否有环，并使用额外的步骤找到环的起始节点。该方法具有 **O(n)** 的时间复杂度和 **O(1)** 的空间复杂度，是解决此类链表问题的经典技巧。

---

### 改进方法 🔧

虽然当前的方法是有效的，但也可以进一步简化一些细节。比如，链表的头部指针不需要虚拟节点，也可以避免使用额外的 `dummy` 节点。

#### 简化的代码实现：

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* slow = head;
        ListNode* fast = head;

        // 快慢指针判断是否存在环
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;

            if (slow == fast) {  // 快慢指针相遇
                // 找到环的起点
                slow = head;
                while (slow != fast) {
                    slow = slow->next;
                    fast = fast->next;
                }
                return slow;  // 返回环的起点
            }
        }

        return NULL;  // 如果没有环
    }
};
```

### 改进方法说明：
- **简化代码**：我们直接用 `head` 作为链表的起始节点，不需要额外的虚拟节点。
- **清晰性**：删除了不必要的代码部分，使得逻辑更加直接。

---

### 改进后的优点：
- **更简洁**：代码结构更加简洁，去除了虚拟节点的创建和使用。
- **逻辑清晰**：代码更容易理解，直接通过链表头节点进行遍历。

