# 🌲 Prim 算法模板：最小生成树 MST（基于优先队列）

---

## 📌 题目背景

给定一个无向图，包含 `V` 个顶点和 `E` 条加权边。  
目标是构造一棵最小生成树（MST），满足：

- 连接所有顶点
- 总边权最小
- 无环

适合使用 **Prim** 算法实现，特别适用于稠密图。

---

## 🚀 Prim 算法核心思想

> 基于“贪心 + 最小堆”的方式逐步扩展当前的连通块

1. 从任意点开始，将其加入 MST
2. 每次从“当前连通块”出发，选择 **最小边** 扩展到新点
3. 重复此过程直到所有节点都加入 MST

---

## ⚙️ 优先队列结构定义

使用 `priority_queue` 来维护当前可扩展边中的最小值：

```cpp
struct cmp {
    bool operator()(pair<int, int> &a, pair<int, int> &b){
        return a.second > b.second; // 小根堆
    }
};
```

---

## 💻 完整代码（C++）

```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

struct cmp {
    bool operator()(pair<int, int> &a, pair<int, int> &b) {
        return a.second > b.second;
    }
};

int main() {
    int V, E;
    cin >> V >> E;

    // 邻接表：grid[u] 存储 {v, 权重}
    vector<vector<pair<int, int>>> grid(V + 1);

    priority_queue<pair<int, int>, vector<pair<int, int>>, cmp> pq;

    for (int i = 0; i < E; i++) {
        int v1, v2, val;
        cin >> v1 >> v2 >> val;
        grid[v1].push_back({v2, val});
        grid[v2].push_back({v1, val});
    }

    int ans = 0;
    vector<bool> visited(V + 1, false);
    visited[1] = true; // 从点 1 开始
    int cntNode = 1;

    for (int i = 0; i < V; i++) {
        // 将当前节点能到的边加入小根堆
        for (auto &edge : grid[cntNode]) {
            pq.push(edge);
        }

        // 跳过已访问节点
        while (!pq.empty() && visited[pq.top().first]) {
            pq.pop();
        }

        if (pq.empty())
            break;

        // 加入最小权边
        auto temp = pq.top(); pq.pop();
        ans += temp.second;
        visited[temp.first] = true;
        cntNode = temp.first;
    }

    cout << ans << endl;

    return 0;
}
```

---

## 📈 时间复杂度分析

- 每条边最多入堆一次：`O(E log E)`
- 每个点访问一次：`O(V)`
- 整体复杂度约为：`O(E log E)`，适合稠密图

---

## 🧠 Prim vs Kruskal

| 算法     | 数据结构     | 适用场景   |
|----------|--------------|------------|
| Prim     | 最小堆       | 稠密图     |
| Kruskal  | 并查集 + 堆  | 稀疏图     |

---

## 🧪 示例输入输出

```
输入：
4 5
1 2 1
1 3 4
2 3 2
2 4 3
3 4 5

输出：
6
```

说明：选中边为 (1-2)=1, (2-3)=2, (2-4)=3，共 6。

---

## 🧾 小结

Prim 算法适合**构造最小生成树（MST）**：

- 每次扩展当前连通块
- 依赖小根堆优化选边效率
- 实现简单，适合稠密图

---

📌 推荐作为 MST 必备模板之一，搭配 Kruskal 一起掌握。
