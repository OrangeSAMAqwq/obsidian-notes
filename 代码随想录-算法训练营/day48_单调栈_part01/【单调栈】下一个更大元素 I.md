# 题目：下一个更大元素 I（Next Greater Element I）🔍📈🧩

---

## 🧠 题目理解

给定两个 **没有重复元素** 的数组 `nums1` 和 `nums2`，其中：

- `nums1` 是 `nums2` 的子集（顺序不一定连续）
- 你需要为 `nums1` 中的每个元素 `x` 找出它在 `nums2` 中对应的下一个更大元素

返回一个等长数组 `ans`，如果 `x` 在 `nums2` 中右边没有更大的元素，则 `ans[i] = -1`

---

## 🔍 问题建模：单调栈 + 哈希表（经典 NGE 模板）

本质是：

> 对于 `nums2` 中的每个元素，找到它右边第一个比它大的元素  
> 然后把这个信息用哈希表记录下来，供 `nums1` 查用

---

## ✨ 单调栈设计（倒序+递减）

处理 `nums2` 时使用一个 **单调递减栈**（栈顶最小）：

- 从右往左扫描 `nums2`
- 维护栈中元素始终为“候选的更大元素”
- 如果当前元素比栈顶大或相等 → 弹出（无意义）
- 如果栈不为空 → 栈顶就是下一个更大元素
- 如果栈为空 → 没有更大元素，返回 -1

用哈希表记录：

```text
hashmap[x] = 下一个更大的元素（或 -1）
```

---

## 🔁 匹配 nums1（O(1) 查表）

处理完 `nums2` 后，直接遍历 `nums1`，将 `nums1[i]` 映射到 `hashmap[nums1[i]]` 即可。

---

## 💻 代码实现（你的版本）

```c++
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int> hashmap;
        vector<int> st;
        int n = nums2.size();
        hashmap[nums2[n - 1]] = -1;
        st.push_back(nums2[n - 1]);
        for (int i = n - 2; i >= 0; i--){
            while (!st.empty() && nums2[i] >= st.back()){
                st.pop_back();
            }
            if (st.empty()){
                hashmap[nums2[i]] = -1;
            }
            else{
                hashmap[nums2[i]] = st.back();
            }
            st.push_back(nums2[i]);
        }

        vector<int> ans;
        for (int i = 0; i < nums1.size(); i++){
            ans.push_back(hashmap[nums1[i]]);
        }
        return ans;
    }
};
```

---

## ⏱️ 复杂度分析

- **时间复杂度**：O(n + m)  
  - 遍历 `nums2` 构建栈 O(n)  
  - 遍历 `nums1` 查表 O(m)
- **空间复杂度**：O(n)  
  - 栈和哈希表都与 `nums2` 大小有关

---

## 🧾 核心记忆点（非常重要）

```text
“下一个更大元素” → 单调递减栈
栈中存的是未来的候选者（右边更大的值）
遍历时记录映射，供查询子集使用
```

一句话总结：

> **用栈预处理 nums2 的每个元素的“下一个更大元素”，再用哈希映射快速回答 nums1 的查询。**

---

## 🔗 思维迁移

这个问题是单调栈经典题之一，其思路在多个题型中高频出现，包括：

- 下一个更大元素 II（环形）
- 柱状图中最大矩形
- 每日温度
- 股票跨度问题

本题的技巧点：

- “预处理大数组，查询小数组”
- 哈希表 + 单调栈组合是处理这种“跨数组映射查询”的常见模式
