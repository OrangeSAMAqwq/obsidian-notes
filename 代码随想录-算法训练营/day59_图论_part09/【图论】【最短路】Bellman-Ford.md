# 📍 Bellman-Ford 算法：单源最短路径（带负权边）🚀

---

## 🧩 题意抽象

给定一个加权有向图，有 `n` 个节点和 `m` 条边，每条边有一个权重 `v`。

目标是从源点 `1` 到目标点 `n` 计算 **最短路径**。

如果目标点无法到达，输出 `"unconnected"`。

---

## 💡 Bellman-Ford 算法核心思想

**Bellman-Ford 算法** 是 **最短路径问题** 的经典算法之一，特别适用于带有负权边的图。它的核心思想是：

> 每次遍历所有边，逐步放松最短路径信息，最多进行 `n - 1` 次遍历。

- **放松操作**：如果从节点 `u` 到节点 `v` 的边 `u -> v` 可以提供一个更短的路径，则更新 `v` 的最短路径。

---

## 🔍 算法设计

### ① 初始化

- `minDist[i]`：表示从源点 `1` 到节点 `i` 的当前最短路径
  - 初始化为 `INT_MAX`（表示不可达）
  - `minDist[1] = 0`（源点到源点的距离为 0）

### ② 放松操作（Relax）

对每一条边 `(s, t, v)`：

- 如果 `minDist[s] != INT_MAX`，则尝试通过 `s` 到 `t` 更新 `t` 的最短路径：

```text
minDist[t] = min(minDist[t], minDist[s] + v)
```

### ③ 最多进行 `n - 1` 次放松

因为图中最多有 `n - 1` 条边，所以最多进行 `n - 1` 次迭代。

### ④ 判断是否存在负权环

- 如果经过 `n - 1` 次迭代后，仍然能够放松某条边，说明图中存在负权环。

---

## ✅ C++ 实现代码

```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

int main(){
    int n, m;
    cin >> n >> m;
    vector<int> minDist(n + 1, INT_MAX);
    minDist[1] = 0;
    vector<vector<int>> grid;
    
    // 读取图的边
    for (int i = 0; i < m; i++){
        int s, t, v;
        cin >> s >> t >> v;
        grid.push_back({s, t, v});
    }

    // Bellman-Ford 放松操作
    bool flag = false;
    for (int i = 1; i < n; i++){
        bool flag = true;
        for (int j = 0; j < grid.size(); j++){
            vector<int>& cnt = grid[j];
            if (minDist[cnt[0]] != INT_MAX){
                minDist[cnt[1]] = min(minDist[cnt[1]], minDist[cnt[0]] + cnt[2]);
                flag = false;
            }
        }
        if (flag)
            break;
    }

    // 检查目标点是否可达
    if (minDist[n] == INT_MAX)
        cout << "unconnected" << endl;
    else
        cout << minDist[n] << endl;

    return 0;
}
```

---

## ⏱️ 复杂度分析

- **时间复杂度**：`O(n * m)`  
  每次遍历所有边，最多执行 `n - 1` 次
- **空间复杂度**：`O(n + m)`  
  存储最短路径数组和图的边信息

---

## 💡 总结

- Bellman-Ford 算法适用于含有 **负权边** 的图，能在最多 `n - 1` 次迭代中找到 **最短路径**。
- 该算法 **能够检测负权环**，但性能不如 Dijkstra 算法（只适用于非负权边）。

---

📌 **一句话记忆**：

> Bellman-Ford = 每次遍历所有边，进行 `n - 1` 次放松，适用于带负权边的最短路径问题。

