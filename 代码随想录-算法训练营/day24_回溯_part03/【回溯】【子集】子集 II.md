# 题目：子集 II (Subsets II) 🧩🌿✨

## 题目链接  
https://leetcode.cn/problems/subsets-ii/

---

## 🧠 解题思路

### 📋 题意理解  
给定一个整数数组 `nums`，其中**可能包含重复元素**，返回所有可能的子集（幂集），并且结果中**不能包含重复子集**。

与「子集 I」相比，唯一变化：

- `nums` 允许重复  
- 需要把重复子集去掉 ✅

---

## 🌟 核心思想：回溯 + 排序 + 树层去重

### 1) 先排序  
排序后，相同元素会相邻，便于识别重复：

```cpp
sort(nums.begin(), nums.end());
```

### 2) 子集收集方式不变  
子集题的特点是：**每个递归节点都是一个合法子集**  
因此每次进入 `dfs` 都先：

```cpp
ans.push_back(t);
```

### 3) 去重关键：树层去重（同一层不选相同元素开头）

如果在同一层中出现相同元素，例如：

```
[1, 2, 2]
```

当 `start` 固定时，如果既选第一个 `2` 开头，又选第二个 `2` 开头，会产生重复子集。

因此加入去重判断：

```cpp
if (i != start && nums[i] == nums[i - 1])
    continue;
```

含义：

- `i != start`：保证只在“同一层”进行去重
- `nums[i] == nums[i-1]`：相同元素在同层只用第一个

✅ 这样可以避免重复子集  
✅ 但允许在不同层使用重复值（例如 `[2,2]` 这种子集仍然合法）

---

## 💻 代码实现

```cpp
class Solution {
public:
    void dfs(auto &ans, auto &t, const auto &nums, int start){
        ans.push_back(t);
        for (int i = start; i < nums.size(); i++){
            if (i != start && nums[i] == nums[i - 1])
                continue;
            t.push_back(nums[i]);
            dfs(ans, t, nums, i + 1);
            t.pop_back();
        }
    }
    
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> ans;
        vector<int> t;
        dfs(ans, t, nums, 0);
        return ans;
    }
};
```

---

## 🔍 关键点总结

### ✅ 与「子集 I」的区别

| 题目 | 数组是否可能重复 | 是否需要去重 | 去重方式 |
|------|------------------|--------------|----------|
| Subsets | ❌ 不重复 | ❌ 不需要 | - |
| Subsets II | ✅ 可能重复 | ✅ 需要 | **排序 + 树层去重** |

---

## ⏱️ 复杂度分析

设 `n = nums.size()`：

- 子集数量上界仍是 `2^n`（重复元素会让实际数量变少）

| 类型 | 复杂度 |
|------|--------|
| 时间复杂度 | O(n · 2^n) |
| 空间复杂度 | O(n)（递归栈 + 当前路径） |

---

## 🧾 总结

- 子集问题：**每个节点都收集答案**
- 有重复元素时：必须 **排序 + 树层去重**
- 去重条件 `i != start` 是关键，它区分了：
  - 同层重复（要去掉）
  - 不同层重复（允许出现，如 `[2,2]`）

```text
子集 II 的去重核心：
同一层不能用同值开头；不同层可以继续用同值扩展
```
