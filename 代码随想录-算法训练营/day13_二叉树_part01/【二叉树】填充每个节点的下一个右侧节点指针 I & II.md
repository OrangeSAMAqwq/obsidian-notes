# 题目：填充每个节点的下一个右侧节点指针（Populating Next Right Pointers in Each Node）🌲➡️

## 题目链接  
[LeetCode - 116. 填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)

---

## 🧠 解题思路

题目要求：  
给定一棵**二叉树**，将同一层的所有节点通过 `next` 指针连接起来，使每个节点的 `next` 指针指向其**右侧的节点**。如果右侧没有节点，则应为 `NULL`。

本题可以通过**层序遍历**（BFS）轻松完成，只需要记录每一层的节点，然后依次连接 `next` 指针即可。

---

## ✅ 解法：BFS 层序遍历 + 指针连接

### 步骤说明：

1. 使用队列 `queue<Node*>` 进行层序遍历；
2. 每次遍历当前层的所有节点（根据 `queue.size()` 确定个数）；
3. 对当前层的每个节点：
   - 若不是该层最后一个节点，则将其 `next` 指向队列的下一个节点；
   - 同时将其子节点（`left` 和 `right`）入队；
4. 最后一层的节点自然不会再被连接，`next` 保持为 `NULL`。

---

## 💻 代码实现

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        if (!root)
            return root;

        std::queue<Node*> q;
        q.push(root);

        while (!q.empty()) {
            int n = q.size();
            for (int i = 0; i < n; i++){
                Node* cnt = q.front();
                q.pop();

                // 不是当前层的最后一个节点
                if (i != n - 1)
                    cnt->next = q.front();

                // 子节点入队
                if (cnt->left)
                    q.push(cnt->left);
                if (cnt->right)
                    q.push(cnt->right);
            }
        }

        return root;
    }
};
```

---

## 🔍 边界条件分析

- 若树为空，则直接返回 `nullptr`；
- 若只有一个节点，`next` 默认就是 `NULL`；
- 该写法**适用于任意二叉树**（非必须是完美二叉树）。

---

## ⏱️ 复杂度分析

| 复杂度类型 | 说明 |
|------------|------|
| 时间复杂度 | O(n)，每个节点仅访问一次 |
| 空间复杂度 | O(n)，最坏情况下队列存储一整层的节点数量 |

---

## 📌 拓展：完美二叉树的 O(1) 空间解法

如果明确给定的是**完美二叉树**（每个非叶子节点都有两个子节点，所有叶子节点在同一层），可以使用 **常数空间解法**：

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        if (!root) return nullptr;

        Node* leftmost = root;
        while (leftmost->left) {
            Node* head = leftmost;
            while (head) {
                head->left->next = head->right;
                if (head->next)
                    head->right->next = head->next->left;
                head = head->next;
            }
            leftmost = leftmost->left;
        }

        return root;
    }
};
```

这种解法通过已经建立好的 `next` 指针逐层向下，不使用队列，空间复杂度为 O(1)。

---

## 🧾 总结

- 使用层序遍历 BFS 可以直观、稳健地处理任意二叉树；
- 若额外限制是**完美二叉树**，可进一步优化到 O(1) 空间；
- 本题是树结构与队列结合的经典题型，非常适合做为 BFS 的模板练习。

```cpp
// 推荐记忆模板
while (!q.empty()) {
    int n = q.size();
    for (int i = 0; i < n; i++) {
        Node* node = q.front();
        q.pop();
        // 处理 node
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```
