# 题目：打家劫舍 III（House Robber III）🌳💰

---

## 🧠 题目理解

这是 **打家劫舍系列的第三题**，结构从「数组」升级成了「二叉树」。

规则依然不变：

- 如果偷了某个节点（房子）
- 就 **不能偷它的直接子节点**
- 求能偷到的 **最大金额**

⚠️ 注意：  
这里的“相邻”不再是数组下标，而是 **父子节点关系**

---

## 🔍 问题建模：树形 DP

这道题是一个非常标准、非常重要的 **树形 DP（后序遍历）** 模型。

核心思想：

> **每个节点，都要同时考虑「偷」和「不偷」两种状态**

---

## ✨ 状态设计（关键点）

对每一个节点 `root`，返回一个长度为 2 的数组：

```text
res[0]：偷当前节点 root 能获得的最大金额
res[1]：不偷当前节点 root 能获得的最大金额
```

这是整道题的灵魂。

---

## 🔁 状态转移分析

设：

- 左子树返回：`leftTree = {偷左, 不偷左}`
- 右子树返回：`rightTree = {偷右, 不偷右}`

---

### 情况一：偷当前节点

如果偷 `root`，**左右孩子都不能偷**：

```text
res[0] = root->val
       + leftTree[1]
       + rightTree[1]
```

---

### 情况二：不偷当前节点

如果不偷 `root`，左右孩子可以 **自由选择偷或不偷**：

```text
res[1] = max(leftTree[0], leftTree[1])
       + max(rightTree[0], rightTree[1])
```

---

## 🔄 遍历方式

- 使用 **后序遍历**
- 先拿到左右子树的状态
- 再计算当前节点的状态

这是一个 **自底向上** 的 DP。

---

## 💻 代码实现（你的版本）

```cpp
class Solution {
public:
    vector<int> robTree(TreeNode *root){
        if (root == nullptr)
            return {0, 0};

        vector<int> leftTree = robTree(root->left);
        vector<int> rightTree = robTree(root->right);

        vector<int> cntNode(2);
        // 偷当前节点：左右子节点都不能偷
        cntNode[0] = root->val + leftTree[1] + rightTree[1];

        // 不偷当前节点：左右子节点可偷可不偷
        cntNode[1] = max(leftTree[0], leftTree[1])
                   + max(rightTree[0], rightTree[1]);

        return cntNode;
    }

    int rob(TreeNode* root) {
        vector<int> ans = robTree(root);
        return max(ans[0], ans[1]);
    }
};
```

---

## 🆚 与前两题的关系

| 题目 | 结构 | DP 形态 |
|----|----|----|
| 打家劫舍 I | 数组 | 线性 DP |
| 打家劫舍 II | 环形数组 | 拆成两个线性 DP |
| 打家劫舍 III | 二叉树 | 树形 DP |

一句话总结：

> **打家劫舍 III = 打家劫舍 I 的“树结构版本”**

---

## ⏱️ 复杂度分析

- 时间复杂度：`O(n)`  
  每个节点只访问一次
- 空间复杂度：`O(h)`  
  递归栈深度，`h` 为树高

---

## 🧾 核心记忆点（非常重要）

```text
树形 DP：
每个节点返回两个状态
[偷当前，不偷当前]
```

```text
偷当前 → 子节点只能“不偷”
不偷当前 → 子节点自由选择
```

这是 **树形 DP 的经典模板之一**，后面很多题都会反复用到。

---

## 🔗 思维迁移

这个模型可以直接迁移到：

- 树上最大独立集
- 树上状态选择类 DP
- 各类「父子互斥」的问题

到这里为止，  
**打家劫舍三部曲你已经全部吃透了** ✅  
而且是非常扎实的那种 💪
