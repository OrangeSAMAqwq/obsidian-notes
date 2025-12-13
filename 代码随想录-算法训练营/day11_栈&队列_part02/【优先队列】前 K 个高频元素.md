# 题目：前 K 个高频元素 (Top K Frequent Elements) 📊🔥

## 题目链接 🌐  
https://leetcode.cn/problems/top-k-frequent-elements/

---

## 解题思路 🧠

### 题意理解 📋

给定一个整数数组 `nums` 和一个整数 `k`，要求返回出现频率 **前 k 高** 的元素，结果顺序不作要求。

例如：

- 输入：`nums = [1,1,1,2,2,3], k = 2`
- 输出：`[1,2]`

---

## 解法思路：哈希表 + 小根堆（优先队列） 🧩

这是一个典型的 **Top K 问题**，核心思想是：

> **用哈希表统计频率，用大小为 k 的小根堆维护当前前 k 高频元素**

### 具体步骤：

1. **统计频率**  
   使用 `unordered_map<int, int>` 统计每个数字在数组中出现的次数。

2. **维护一个小根堆**  
   - 堆中存放 `(元素值, 出现次数)`  
   - 按照「出现次数」排序  
   - 堆顶是当前 **频率最小** 的元素  

3. **遍历哈希表并入堆**  
   - 每加入一个元素到堆中  
   - 若堆大小超过 `k`，则弹出堆顶（频率最小的元素）  
   - 保证堆中始终只保存 **频率最高的 k 个元素**

4. **取出结果**  
   - 最终堆中剩下的 `k` 个元素即为答案

---

## 代码实现 💻

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::unordered_map<int, int> hashmap;
        vector<int> ans;

        // 1. 统计频率
        for (int i = 0; i < nums.size(); i++){
            hashmap[nums[i]]++;
        }

        // 2. 定义小根堆（按出现次数排序）
        std::priority_queue<
            pair<int, int>,
            vector<pair<int, int>>,
            function<bool(pair<int, int>, pair<int, int>)>
        > pq(
            [](pair<int, int> a, pair<int, int> b) {
                return a.second > b.second; // second 小的优先级更高
            }
        );

        // 3. 维护大小为 k 的小根堆
        for (auto it = hashmap.begin(); it != hashmap.end(); it++){
            pq.push({it->first, it->second});
            if (pq.size() > k)
                pq.pop();
        }

        // 4. 收集结果
        for (int i = 0; i < k; i++){
            ans.push_back(pq.top().first);
            pq.pop();
        }

        return ans;
    }
};
```

---

## 关键点 🔑

- **unordered_map**：用于 O(1) 平均时间统计元素频率；
- **priority_queue（小根堆）**：
  - 堆顶始终是当前频率最小的元素；
  - 通过限制堆大小为 `k`，实现高效筛选；
- 使用 **lambda 比较函数** 自定义堆的排序规则。

---

## 边界情况 🚨

1. `k == 1`：返回出现次数最多的那个元素；
2. `k == nums.size()`：返回所有不重复元素；
3. `nums` 中只有一种元素；
4. 多个元素出现频率相同：题目允许任意顺序返回。

---

## 复杂度分析 ⏱️

- 时间复杂度：**O(n log k)**
  - 统计频率 O(n)
  - 每次堆操作 O(log k)
- 空间复杂度：**O(n + k)**
  - 哈希表 O(n)
  - 堆最多存 k 个元素

---

## 总结 📚

这道题是 **哈希表 + 堆** 的经典组合题：

- 哈希表解决「统计问题」
- 堆解决「Top K 问题」
- 使用 **小根堆 + 限制大小**，在保证正确性的同时显著降低时间复杂度

这套思路在：
- Top K 频率
- Top K 最大 / 最小值
- 流式数据统计  
等场景中都非常通用，值得熟练掌握。
