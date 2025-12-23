# 题目：组合总和 III (Combination Sum III) 🎯➕🔢

## 题目链接  
https://leetcode.cn/problems/combination-sum-iii/

---

## 🧠 解题思路

### 📋 题意理解  
从 `1 ~ 9` 中选出 **恰好 `k` 个不同数字**，使它们的和等于 `n`，返回所有满足条件的组合。

约束点很明确：

- 只能用 `1 ~ 9`
- 每个数字最多用一次（不可重复）
- 必须选 **k 个数**
- 和必须等于 **n**
- 返回的是“组合”（顺序无关）

---

## 🌟 核心思想：回溯（Backtracking）+ 剪枝

使用回溯枚举所有可能的组合路径 `t`：

- `t`：当前选择的数字集合
- `sum`：当前路径的累加和
- `cnt`：下一次可选数字的起点（保证不重复、递增）

当满足：

- `t.size() == k` 且 `sum == n`

则记录答案。

---

## ✅ 剪枝策略（非常关键）

在回溯过程中，如果出现以下情况，可以直接返回：

- `sum > n`：和已经超了，不可能再变回去
- `t.size() > k`：选的数字超过 k 个了

这些剪枝能显著减少搜索空间 ✂️

---

## ✨ 递归函数设计

```cpp
dfs(ans, t, k, n, cnt, sum)
```

参数含义：

| 参数 | 含义 |
|------|------|
| `ans` | 最终答案集合 |
| `t` | 当前组合路径 |
| `k` | 目标选取数量 |
| `n` | 目标和 |
| `cnt` | 当前层可选择的起始数字 |
| `sum` | 当前路径和（引用传递，便于加减回溯） |

---

## 💻 代码实现

```cpp
class Solution {
public:
    void dfs(vector<vector<int>> &ans, vector<int> &t, int k, int n, int cnt, int &sum){
        if (sum == n && t.size() == k){
            ans.push_back(t);
            return;
        }
        if (sum > n || t.size() > k)
            return;

        for (int i = cnt; i <= 9; i++){
            sum += i;
            t.push_back(i);

            dfs(ans, t, k, n, i + 1, sum);

            sum -= i;
            t.pop_back();
        }
    }

    vector<vector<int>> combinationSum3(int k, int n) {
        vector<vector<int>> ans;
        vector<int> t;
        int sum = 0;
        dfs(ans, t, k, n, 1, sum);
        return ans;
    }
};
```

---

## 🔍 关键点解析

- `cnt` 保证组合递增，从而避免重复：
  - 选了 `i` 后下一层从 `i+1` 开始
- `sum` 通过引用传递，并在回溯时加回去 / 减回去
- 先判断成功条件，再做剪枝，逻辑清晰

---

## ⏱️ 复杂度分析

搜索空间最大为从 9 个数里选 k 个：

- 时间复杂度：  
  \[
  O(C(9, k))
  \]
  由于有剪枝，实际会更快

- 空间复杂度：  
  - 递归深度：`O(k)`
  - 路径数组：`O(k)`
  - 输出答案占用：取决于结果数量

---

## 🧾 总结

- 本题是「组合回溯」+「目标和约束」的典型模板题
- 核心是三件事：
  1. 递增选择（避免重复）
  2. 维护路径和 `sum`
  3. 剪枝（`sum>n` / `size>k`）

```text
组合类回溯 = for 枚举选择 + 递归深入 + 回溯撤销 + 剪枝提速
```

这题写顺后，很多组合/子集/剪枝题都会轻松很多 👍
