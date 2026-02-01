# 🌉 Floyd-Warshall 算法模板笔记：任意两点间的最短路径

---

## 🧠 问题描述

给定一个无向图，包含 `N` 个节点、`M` 条边。接下来有 `Q` 个查询，每次询问两个点之间的最短路径距离。

### 输入格式：

- 第一行：`N M`，表示点数和边数
- 接下来的 `M` 行：每行 `u v w`，表示一条边 `u - v`，权重为 `w`（无向图）
- 然后一个整数 `Q`，表示查询数量
- 接下来 `Q` 行：每行 `start end`，询问从 `start` 到 `end` 的最短距离

---

## 🔧 解法：Floyd-Warshall 多源最短路径算法

> Floyd-Warshall 是一种 **动态规划** 算法，用于求任意两点之间的最短路径。

### ✅ 核心思想：

对于任意三元组 `(i, j, k)`：

```text
minDist[i][j] = min(minDist[i][j], minDist[i][k] + minDist[k][j])
```

含义：若从 `i` 到 `j` 经过 `k` 更短，则更新。

---

## 📦 数据结构设计

- `minDist[i][j]`：表示从点 `i` 到点 `j` 的最短路径
- 初始值：
  - `i == j` 时为 0（自己到自己）
  - 其余为 `INT_MAX`（表示不可达）
- 遍历顺序：三重循环，**最外层枚举中转点 k**

---

## 💻 代码实现

````cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

int main() {
    int N, M;
    cin >> N >> M;
    vector<vector<int>> minDist(N + 1, vector<int>(N + 1, INT_MAX));

    // 初始化：自己到自己为 0
    for (int i = 1; i <= N; i++) {
        minDist[i][i] = 0;
    }

    // 输入边信息（无向图）
    for (int i = 0; i < M; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        minDist[u][v] = min(minDist[u][v], w);
        minDist[v][u] = min(minDist[v][u], w);
    }

    // 核心算法：Floyd-Warshall
    for (int k = 1; k <= N; k++) {
        for (int i = 1; i <= N; i++) {
            for (int j = 1; j <= N; j++) {
                if (minDist[i][k] != INT_MAX && minDist[k][j] != INT_MAX) {
                    minDist[i][j] = min(minDist[i][j], minDist[i][k] + minDist[k][j]);
                }
            }
        }
    }

    // 处理 Q 次查询
    int Q;
    cin >> Q;
    for (int i = 0; i < Q; i++) {
        int start, end;
        cin >> start >> end;
        if (minDist[start][end] == INT_MAX)
            cout << -1 << endl;
        else
            cout << minDist[start][end] << endl;
    }

    return 0;
}
````

---

## ⏱️ 复杂度分析

- **时间复杂度**：O(N³)，适用于 N <= 500 的稠密图
- **空间复杂度**：O(N²)

---

## 🧾 总结

- Floyd-Warshall 非常适合 **多源最短路径问题**
- 对于边数较多，或要查询多次点对之间距离的场景，**优于 Dijkstra 多次调用**
- 若有负权边但无负环，也能正确处理

--- 
