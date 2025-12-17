# 题目：路径总和 I & II (Path Sum / Path Sum II) 🧮🌲

## 题目链接 🌐  
- Path Sum（路径总和 I）：https://leetcode.cn/problems/path-sum/  
- Path Sum II（路径总和 II）：https://leetcode.cn/problems/path-sum-ii/

---

## 🧠 核心思路总览

两题的共同点：  
都要求判断/找出 **从根节点到叶子节点** 的路径，使得路径上节点值之和满足目标 `targetSum`。

- ✅ 路径必须是 **root → leaf**
- ✅ 叶子节点定义：左右孩子都为空
- ✅ 都适合用 DFS 递归遍历整棵树

区别在于输出：

| 题目 | 要求 |
|------|------|
| 路径总和 I | 只要判断是否存在一条满足条件的路径（true/false） |
| 路径总和 II | 需要返回所有满足条件的路径（路径列表） |

---

# 一、路径总和 I（是否存在）✅

## 解题思路 🧠

使用 DFS 递归累加路径和 `sum`：

- 每到一个节点，把节点值加入 `sum`
- 如果到达叶子节点，检查 `sum == targetSum`
- 如果找到任意一条满足的路径，就用 `flag` 提前结束递归（剪枝）

---

## 代码实现 💻

```cpp
class Solution {
public:
    void dfs(TreeNode* root, int sum, bool &flag, int target){
        if (flag == true)
            return;
        sum += root->val;
        if (root->left == nullptr && root->right == nullptr){
            if (sum == target){
                flag = true;
            }
            return;
        }
        if (root->left)
            dfs(root->left, sum, flag, target);
        if (root->right)
            dfs(root->right, sum, flag, target);
    }

    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root)
            return false;
        bool flag = false;
        dfs(root, 0, flag, targetSum);
        return flag;
    }
};
```

---

## 关键点 🔑

- `flag` 作为全局剪枝：找到就停，避免无意义遍历
- 只在 **叶子节点** 判断是否等于目标和

---

# 二、路径总和 II（输出所有路径）🧾✅

## 解题思路 🧠

同样 DFS，但需要额外维护：

- `t`：当前路径（vector）
- `cnt`：当前路径和
- `ans`：所有符合条件的路径集合

做法：

1. 进入节点：`t.push_back(root->val)`，`cnt += root->val`
2. 到叶子节点：
   - 若 `cnt == target`，将 `t` 加入 `ans`
3. 回溯：
   - 遍历完左/右子树后，需要 `t.pop_back()` 恢复路径状态

---

## 代码实现 💻

```cpp
class Solution {
public:
    void dfs(auto &ans, auto &t, TreeNode* root, int cnt, int target) {
        cnt += root->val;
        t.push_back(root->val);

        if (root->left == nullptr && root->right == nullptr){
            if (cnt == target)
                ans.push_back(t);
            return;
        }

        if (root->left){
            dfs(ans, t, root->left, cnt, target);
            t.pop_back();
        }
        if (root->right){
            dfs(ans, t, root->right, cnt, target);
            t.pop_back();
        }
    }

    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        vector<vector<int>> ans;
        if (!root)
            return ans;
        vector<int> t;
        dfs(ans, t, root, 0, targetSum);
        return ans;
    }
};
```

---

## 关键点 🔑

- `t` 是共享路径容器，所以必须 **回溯 pop_back**
- 只在 **叶子节点** 判断并加入答案
- 注意：这里在叶子节点 `return` 前没有 `pop_back`，但因为父节点在递归返回后会 `pop_back()`，整体仍能恢复路径状态（写法偏“父节点负责回溯”）

---

## ⚠️ 易错点总结

1. **必须是根到叶子**：中间节点不能算路径终点  
2. **回溯必须正确**：否则路径会串到别的分支  
3. Path Sum I 可以剪枝，Path Sum II 不能随便剪枝（要找全）  

---

## ⏱️ 复杂度分析

设节点数为 `n`，树高为 `h`：

### 路径总和 I
- 时间复杂度：O(n)（最坏遍历整棵树）
- 空间复杂度：O(h)（递归栈）

### 路径总和 II
- 时间复杂度：O(n) ~ O(n * h)（拷贝路径到答案时有额外开销）
- 空间复杂度：O(h) + 输出答案占用

---

## 🧾 总结

- 两题本质都是 DFS + 根到叶路径处理
- I：判断存在即可，适合 `flag` 剪枝 ✅
- II：要收集所有路径，重点在回溯与路径记录 🧾

这两题可以作为「树的路径类题目」核心模板，后续很多题都是这个形态的变种。
