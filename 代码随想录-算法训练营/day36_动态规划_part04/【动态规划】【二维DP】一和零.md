# 题目：一和零 (Ones and Zeroes) 🔢🎒

## 题目链接  
https://leetcode.cn/problems/ones-and-zeroes/

---

## 🧠 解题思路

### 📋 题意理解  

给定一个字符串数组 `strs`，每个字符串只包含 `'0'` 和 `'1'`。

同时给定两个整数：

- `m`：最多能使用的 `0` 的数量
- `n`：最多能使用的 `1` 的数量

问：**在不超过 `m` 个 0、`n` 个 1 的前提下，最多能选多少个字符串？**

⚠️ 每个字符串 **只能选一次**。

---

## 🌟 核心思想：二维 01 背包（双容量约束）

这是一个非常经典、也非常有代表性的 DP 题：

> **01 背包 + 两个容量维度 + 最大数量**

可以把问题抽象成：

- 每个字符串 = 一个物品
- 物品消耗：
  - `countZero` 个 0
  - `countOne` 个 1
- 物品价值 = `1`（选了一个字符串）
- 背包容量：
  - 0 的容量 = `m`
  - 1 的容量 = `n`

---

## ✨ 状态设计

### ✅ 状态定义

```text
dp[j][k] = 在最多使用 j 个 0、k 个 1 的情况下，最多能选的字符串数量
```

---

### ✅ 初始化

```cpp
dp[j][k] = 0
```

表示在什么都不选的情况下，选中字符串数量为 0。

---

### ✅ 状态转移方程（01 背包）

对于每个字符串，先统计它消耗的资源：

```text
countZero = 字符串中 '0' 的数量
countOne  = 字符串中 '1' 的数量
```

如果选择当前字符串：

```text
dp[j][k] = max(
    dp[j][k],
    dp[j - countZero][k - countOne] + 1
)
```

---

## ⚠️ 关键点：双重倒序遍历

```cpp
for (int j = m; j >= countZero; j--)
    for (int k = n; k >= countOne; k--)
```

### 为什么必须倒序？

- 每个字符串只能用一次（01 背包）
- 倒序保证：
  - 当前字符串不会在同一轮被重复使用

这是 **01 背包的铁律**，即使是二维容量也一样。

---

## 💻 代码实现

```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

        for (int i = 0; i < strs.size(); i++){
            int countZero = 0, countOne = 0;
            for (int j = 0; j < strs[i].size(); j++){
                if (strs[i][j] == '0')
                    countZero++;
                else
                    countOne++;
            }

            for (int j = m; j >= countZero; j--){
                for (int k = n; k >= countOne; k--){
                    dp[j][k] = max(dp[j][k],
                                   dp[j - countZero][k - countOne] + 1);
                }
            }
        }
        return dp[m][n];
    }
};
```

---

## 🔍 关键点总结

- 本题是 **01 背包的进阶版**
- 特点在于：
  - 两个容量维度（0 和 1）
  - 价值统一为 1（选中字符串数量）
- 状态设计清晰，转移逻辑标准
- 双重倒序遍历是核心中的核心

---

## ⏱️ 复杂度分析

设：
- `L = strs.size()`
- `m` 为 0 的容量
- `n` 为 1 的容量

| 类型 | 复杂度 |
|------|--------|
| 时间复杂度 | O(L × m × n) |
| 空间复杂度 | O(m × n) |

---

## 🧾 总结

- 这是 **多维 01 背包** 的经典模板题
- 可以和以下题目形成一个完整背包体系：

```text
01 背包（单容量）
→ 子集和
→ 最后一块石头
→ 目标和
→ 一和零（双容量）
```

非常值得在 Obsidian 中单独标注为：

> **DP · 背包问题 · 多维 01 背包（双约束）** 🎒📌
