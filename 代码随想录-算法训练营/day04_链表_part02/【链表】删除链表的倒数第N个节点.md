# 题目：删除链表的倒数第N个节点 (Remove Nth Node From End of List) ❌
## 题目链接 🌐  
[移除链表的倒数第 N 个节点 - LeetCode](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

---

## 解题思路 🧠

### 快慢指针（Two-pointer Technique）⏩

本题要求移除链表中的倒数第 N 个节点。我们可以通过 **快慢指针** 技巧来解决这个问题，快指针先走 N 步，然后快慢指针同步移动，直到快指针到达链表末尾。此时，慢指针指向的节点的下一个节点就是我们要删除的节点。

### 步骤 📋：

1. **虚拟头节点** 🧾：
   - 创建一个虚拟头节点 `dummy`，它指向链表的头部，这样方便处理边界情况（例如删除头节点）。

2. **快慢指针** 🏃‍♂️🏃‍♀️：
   - 使用两个指针 `cnt` 和 `pre`。首先，`count` 指针走 `n` 步，确保它与 `pre` 指针之间有 N 个节点的距离。
   - 然后，快指针和慢指针同步移动，直到快指针到达链表末尾，此时慢指针的下一个节点就是要删除的节点。

3. **删除节点** 🗑️：
   - 通过更新 `pre->next`，跳过 `cnt` 节点，实现删除倒数第 N 个节点的操作。

4. **返回结果** 🏁：
   - 返回 `dummy->next`，即删除节点后的链表头。

---

### 时间复杂度 ⏱️：

- **O(n)**，其中 `n` 是链表的长度。我们只需要一次遍历链表。
- 空间复杂度为 **O(1)**，只使用了常数的额外空间。

---

## 代码实现 💻

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummy = new ListNode();  // 创建虚拟头节点
        dummy->next = head;
        ListNode* cnt = head, *pre = dummy;
        ListNode* count = head;

        // 让 count 指针走 n 步，确保它和 pre 之间有 n 个节点
        for (int i = 0; i < n; i++) {
            if (count == nullptr)  // 防止链表长度小于 n
                return head;
            count = count->next;
        }

        // 同步移动 fast 和 slow 指针，直到 fast 指针到达链表末尾
        while(count) {
            count = count->next;
            pre = pre->next;
            cnt = cnt->next;
        }

        // 删除倒数第 n 个节点
        pre->next = cnt->next;

        // 返回新的链表头
        return dummy->next;
    }
};
```

---

## 关键点 🔑：

- **虚拟头节点**：使用虚拟头节点来处理头节点被删除的特殊情况。
- **快慢指针**：通过快慢指针技术来找到倒数第 N 个节点，避免两次遍历。
- **节点删除**：通过修改 `pre->next` 来删除倒数第 N 个节点。

---

## 边界情况 🚨：

1. **链表长度小于 N** 🛑：如果链表长度小于 N，直接返回原链表。
2. **删除头节点** 🔴：当需要删除头节点时，虚拟头节点可以帮助简化代码逻辑。
3. **链表只有一个节点** 🧑‍💻：如果链表只有一个节点，删除该节点后返回空链表。

---

## 总结 📚

本题通过 **快慢指针** 技巧高效地解决了链表中倒数第 N 个节点的删除问题。利用虚拟头节点简化了代码，并确保了边界情况的处理。时间复杂度为 **O(n)**，空间复杂度为 **O(1)**，是链表类问题中的常见技巧。
