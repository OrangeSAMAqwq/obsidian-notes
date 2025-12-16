# 题目：N 叉树的层序遍历 (N-ary Tree Level Order Traversal) 🌲🔄

## 题目链接 🌐  
https://leetcode.cn/problems/n-ary-tree-level-order-traversal/

---

## 解题思路 🧠

### 题意理解 📋

给定一棵 **N 叉树**，要求返回其节点值的 **层序遍历结果**。即从上到下、从左到右，每层的节点值分别作为一个子数组组成返回结果。

---

## 解法：广度优先遍历（BFS） + 队列 🧩

这道题是标准的 **层序遍历** 问题，延续二叉树层序遍历的思路，只需将每个节点的所有子节点都加入队列即可，无需特殊处理左右节点。

### 算法步骤：

1. 创建一个结果数组 `ans`，用于存储每层的节点值；
2. 初始化一个队列 `queue<Node*>`，将根节点 `root` 加入队列；
3. 当队列不为空时，执行以下操作：
   - 记录当前层节点数量 `n = q.size()`；
   - 遍历这 `n` 个节点，分别：
     - 弹出队首节点；
     - 将其值加入当前层数组；
     - 将其所有非空子节点加入队列；
4. 每一层处理完后，将当前层数组加入结果 `ans`；
5. 返回结果。

---

## 代码实现 💻

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        if (!root)
            return {};

        vector<vector<int>> ans;
        std::queue<Node*> q;
        q.push(root);

        while (!q.empty()) {
            vector<int> level;
            int n = q.size();
            for (int i = 0; i < n; i++) {
                Node* cnt = q.front();
                q.pop();
                level.push_back(cnt->val);

                // 加入当前节点的所有非空子节点
                for (int j = 0; j < cnt->children.size(); j++) {
                    if (cnt->children[j])
                        q.push(cnt->children[j]);
                }
            }
            ans.push_back(level);
        }

        return ans;
    }
};
```

---

## 关键点总结 🔑

- 与二叉树不同，**N 叉树的每个节点有多个子节点**，因此遍历时使用 `for` 遍历 `children`；
- 使用 **队列实现广度优先遍历** 是常规解法；
- 每次遍历队列中的 `n` 个节点，就表示正在访问当前树的一层。

---

## 复杂度分析 ⏱️

- **时间复杂度：O(n)**  
  所有节点都被访问一次。

- **空间复杂度：O(n)**  
  队列中最多存储一整层的节点，最坏情况下为 O(n)。

---

## 总结 📚

这道题是 BFS 层序遍历在 **N 叉树结构上的自然延伸**。掌握该题能帮助你：

- 熟悉任意树结构的遍历；
- 建立“遍历 + 队列”的模板；
- 为后续进阶题打下基础，例如：Z 字形遍历、多叉树最大深度、路径和等变种问题。
