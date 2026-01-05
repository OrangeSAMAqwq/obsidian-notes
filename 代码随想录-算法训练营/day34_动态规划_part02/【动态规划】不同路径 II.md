# 题目：不同路径 II (Unique Paths II) 🚧🧭

## 题目链接  
https://leetcode.cn/problems/unique-paths-ii/

---

## 🧠 解题思路

### 📋 题意理解  

这是「不同路径」的升级版：

- 给定一个 `m × n` 的网格
- `obstacleGrid[i][j] == 1` 表示 **障碍物**
- 机器人：
  - 从左上角 `(0,0)` 出发
  - 只能向右或向下走
- 目标：**在避开障碍物的前提下，到达右下角的路径数**

---

## 🌟 核心思想：二维动态规划（2D DP）

整体思路与 **不同路径 I** 完全一致，只是多了一个条件判断：

> **遇到障碍物，该位置的路径数为 0**

---

## ✨ 状态设计

### ✅ 状态定义

```cpp
grid[i][j] = 到达位置 (i, j) 的不同路径数（避开障碍）
```

---

### ✅ 初始化

- 如果起点或终点是障碍物，直接返回 `0`
- 起点 `(0,0)` 若无障碍：

```cpp
grid[0][0] = 1
```

---

### ✅ 状态转移方程

对于位置 `(i, j)`：

- 如果是障碍物：

```text
grid[i][j] = 0
```

- 否则：

```text
grid[i][j] = grid[i - 1][j] + grid[i][j - 1]
```

⚠️ 注意边界：
- `i == 0` 时，只能从左边来
- `j == 0` 时，只能从上边来

---

## 🔁 遍历顺序

```text
从上到下
从左到右
```

确保每个状态在被使用前已经计算完成。

---

## 💻 代码实现

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size(), n = obstacleGrid[0].size();
        if (obstacleGrid[m - 1][n - 1] == 1 || obstacleGrid[0][0] == 1)
            return 0;

        vector<vector<int>> grid(m, vector<int>(n, 0));
        grid[0][0] = 1;

        for (int i = 0; i < grid.size(); i++){
            for (int j = 0; j < grid[i].size(); j++){
                if (i == 0 && j == 0)
                    continue;
                if (obstacleGrid[i][j] == 1)
                    continue;
                int up = 0, left = 0;
                if (i > 0)
                    up = grid[i - 1][j];
                if (j > 0)
                    left = grid[i][j - 1];
                grid[i][j] = up + left;
            }
        }
        return grid[m - 1][n - 1];
    }
};
```

---

## 🔍 关键点总结

- **障碍物处理 = 直接跳过，不赋值**
- 起点和终点要提前判断
- 相比「不同路径 I」：
  - 核心 DP 模型完全一致
  - 只是多了一层条件判断

---

## ⏱️ 复杂度分析

| 类型 | 复杂度 |
|----|----|
| 时间复杂度 | O(m × n) |
| 空间复杂度 | O(m × n) |

---

## 🧠 可记录的优化思路

和前一题一样：

```text
grid[i][j] 只依赖 grid[i-1][j] 和 grid[i][j-1]
```

可以使用 **一维滚动数组** 优化空间到 O(n)。

但当前实现：

- 状态语义清晰
- 对障碍处理非常直观
- 非常适合作为「二维 DP + 条件约束」模板

---

## 🧾 总结

- 本题是「不同路径」的 **自然进阶**
- 抽象模型不变，只是：
  - **某些状态不可达**
- 这是后续很多题的基础，例如：
  - 最小路径和（带权）
  - 各种带障碍网格 DP

```text
路径 DP = 状态可达性 + 累加规则
```

非常推荐将本题与「不同路径 I」放在一起对比复习，作为  
👉 **DP · 二维 DP · 网格路径模型（进阶）** 📌
