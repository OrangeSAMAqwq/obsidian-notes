# 题目：二叉树的层序遍历 II (Binary Tree Level Order Traversal II) 🪶🌳

## 题目链接 🌐  
https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/

---

## 解题思路 🧠

### 题意理解 📋

给定一棵二叉树，要求**自底向上**地返回每一层的节点值，层与层之间的顺序要保持自上而下的顺序。

例如二叉树：

```
    3
   / \
  9  20
     / \
    15  7
```

返回结果为：

```
[
  [15, 7],
  [9, 20],
  [3]
]
```

---

## 核心思路：层序遍历 + 反转结果 🧩

这道题在层序遍历的基础上，需要返回 **自底向上的层序遍历**。实现方法如下：

1. 使用 **标准的层序遍历**：
   - 使用队列逐层访问二叉树。
   - 将每一层的节点值加入结果中。

2. 反转结果数组：
   - 在完成层序遍历后，将结果数组反转，即可得到自底向上的遍历顺序。

---

## 代码实现 💻

```cpp
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        if (!root)
            return {};  // 空树直接返回空

        vector<vector<int>> ans;
        std::queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()){
            int n = q.size();
            vector<int> cnt;
            for (int i = 0; i < n; i++){
                TreeNode* temp = q.front();
                q.pop();
                cnt.push_back(temp->val);

                // 将左右子节点加入队列
                if (temp->left)
                    q.push(temp->left);
                if (temp->right)
                    q.push(temp->right);
            }
            ans.push_back(cnt);  // 当前层加入结果
        }

        reverse(ans.begin(), ans.end());  // 反转结果数组

        return ans;
    }
};
```

---

## 关键点 🔑

- **层序遍历**：使用队列 `queue<TreeNode*> q` 来处理每一层的节点。
- **反转操作**：在遍历完成后，使用 `reverse` 函数将结果数组反转，得到从底到上的遍历顺序。
- **边界处理**：
  - 空树 `root == nullptr` 时直接返回空数组。

---

## 边界情况 🚨

1. 空树（`root == nullptr`）：返回 `[]`；
2. 只有根节点：返回 `[[root->val]]`；
3. 单侧树（只有左子树或右子树）：仍然按照层序遍历自底向上输出；
4. 极度不平衡的树：保持 `queue` 逐层推进，逻辑不变。

---

## 复杂度分析 ⏱️

- **时间复杂度**：`O(n)`，每个节点被访问一次；
- **空间复杂度**：`O(n)`，队列最多存储树的一层节点，最坏情况下为 `O(n)`。

---

## 总结 📚

这道题是层序遍历的一种变种，要求返回 **自底向上的层序遍历**：

- 通过标准的 **层序遍历** 获得按层分组的节点值；
- 最后对结果进行反转，轻松获得从底到上的遍历顺序；
- 是二叉树题目中基础且重要的变形，值得掌握。

可以将这段代码作为层序遍历的**通用模板**，并根据需求进行扩展，例如：
- 改为**Z字形层序遍历**；
- 获取每层的**最大值**或**最小值**。

