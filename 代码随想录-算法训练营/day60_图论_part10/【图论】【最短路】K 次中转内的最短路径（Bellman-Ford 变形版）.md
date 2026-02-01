# 🚇 算法模板：K 次中转内的最短路径（Bellman-Ford 变形版）

---

## 📌 问题描述

给定一个带权有向图 `grid`，每条边形如：

```
from -> to，代价为 price
```

输入参数：

- `n`：节点总数（1 ~ n）
- `m`：边数
- `src`：起点编号
- `dst`：终点编号
- `k`：最多允许中转次数（即最多经过 `k+1` 条边）

---

## 🧠 算法核心思想

这是 Bellman-Ford 算法的一个经典变形：

> 在最多 `k + 1` 步以内，计算从起点到终点的最短路径。

区别在于：

- **标准 Bellman-Ford**：执行 `n-1` 次松弛
- **此题版本**：最多执行 `k + 1` 次松弛
- 每轮必须使用 **上一轮的状态**（不能边更新边用）

---

## 📚 数据结构设计

- `grid`：边集，形如 `{from, to, price}`
- `minDist`：当前轮的最短路径
- `minDist_copy`：保存上一轮的最短路径结果，避免链式更新

---

## ✅ 模板代码实现

````cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

int main() {
    int src, dst, k, p1, p2, val, m, n;
    cin >> n >> m;

    vector<vector<int>> grid;

    for (int i = 0; i < m; i++) {
        cin >> p1 >> p2 >> val;
        grid.push_back({p1, p2, val});
    }

    cin >> src >> dst >> k;

    vector<int> minDist(n + 1, INT_MAX);
    minDist[src] = 0;
    vector<int> minDist_copy(n + 1);

    for (int i = 1; i <= k + 1; i++) {
        minDist_copy = minDist;
        for (vector<int> &side : grid) {
            int from = side[0];
            int to = side[1];
            int price = side[2];
            if (minDist_copy[from] != INT_MAX && minDist[to] > minDist_copy[from] + price) {
                minDist[to] = minDist_copy[from] + price;
            }
        }
    }

    if (minDist[dst] == INT_MAX)
        cout << "unreachable" << endl;
    else
        cout << minDist[dst] << endl;
}
````

---

## 📏 样例说明

**输入：**
```
5 6
1 2 100
2 3 100
1 3 500
3 4 100
4 5 100
2 5 600
1 5 2
```

表示从 1 出发，最多经过 **2 次中转**，求到 5 的最短路径。

**输出：**
```
400
```

---

## ⏱️ 时间复杂度分析

- **时间复杂度**：`O(k * m)`  
  每轮最多 `m` 条边，执行 `k + 1` 轮松弛
- **空间复杂度**：`O(n)`

---

## 🧾 应用场景

- Leetcode 787（K 次中转内最便宜的航班）
- 类似图中受限制路径问题（限制边数 / 路径长度）

---

## 🧠 总结

该算法是 Bellman-Ford 的一个实际工程变形：

- 适用于边权为正、允许重复点但限制路径长度的问题
- 若不限制中转次数，就可以退化为标准 Bellman-Ford 或 Dijkstra

---
