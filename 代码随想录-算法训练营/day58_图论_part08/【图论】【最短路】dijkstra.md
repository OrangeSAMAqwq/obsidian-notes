# Dijkstra 最短路（优先队列 / 邻接表）🚀🛣️

## 🧩 题意抽象

给定一个带权有向图：

- `N` 个节点（编号 `1 ~ N`）
- `M` 条有向边 `(S -> E)`，边权为 `V`
- 求：从 **1 号点到 N 号点** 的最短路径长度

若不可达输出 `-1`。

---

## 💡 核心思想

本题是经典的 **单源最短路** 模型，并且边权为非负数（Dijkstra 的适用前提）。

Dijkstra 的核心贪心结论：

> 每次从「当前所有可到达的点」中，选择一个距离最小的未确定节点  
> 该节点的最短距离一旦确定，就不会再被更新

实现方式：

- `minDist[i]` 记录从起点到 `i` 的当前最短距离
- 使用小根堆（优先队列）每次取出 **当前距离最小的候选点**
- 用 `visited` 标记某个点的最短路是否已确定

---

## 🔍 状态设计

### ① 邻接表

```text
grid[u] = {(v, w), ...}
```

表示从 `u` 出发能到达 `v`，边权为 `w`。

---

### ② 最短距离数组

```text
minDist[i]：从 1 到 i 的最短距离（当前最优）
```

初始化：

- `minDist[1] = 0`
- 其余为 `INF`

---

### ③ visited 数组

```text
visited[i] = true：i 的最短距离已经确定
```

---

## 🔁 松弛操作（Relax）

对某个已经确定的点 `node`，遍历它的所有出边 `(node -> next, w)`：

如果：

```text
minDist[node] + w < minDist[next]
```

则更新：

```text
minDist[next] = minDist[node] + w
```

并将 `next` 放入堆中作为候选节点。

---

## ✅ 代码实现（你的版本）

```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <queue>
#include <climits>
using namespace std;

struct cmp{
    bool operator()(const pair<int, int> &a, const pair<int, int> &b){
        return a.second > b.second; // 小根堆：距离小的优先
    }
};

void addNode(const auto &grid, auto &pq, int node,
             auto &visited, auto &minDist){
    for (int i = 0; i < grid[node].size(); i++){
        pair<int, int> cnt = grid[node][i];
        if (visited[cnt.first] == true)
            continue;

        cnt.second += minDist[node];
        minDist[cnt.first] = min(minDist[cnt.first], cnt.second);
        pq.push(cnt);
    }
}

int main(){
    int N, M;
    cin >> N >> M;

    vector<vector<pair<int, int>>> grid(N + 1);
    priority_queue<pair<int, int>, vector<pair<int, int>>, cmp> pq;

    for (int i = 0; i < M; i++){
        int S, E, V;
        cin >> S >> E >> V;
        grid[S].push_back({E, V}); // 有向边
    }

    vector<int> minDist(N + 1, INT_MAX);
    vector<bool> visited(N + 1, false);

    minDist[1] = 0;
    visited[1] = true;

    addNode(grid, pq, 1, visited, minDist);

    while (!pq.empty()){
        pair<int, int> temp = pq.top();
        pq.pop();

        int node = temp.first;
        if (visited[node] == true)
            continue;

        visited[node] = true;
        addNode(grid, pq, node, visited, minDist);
    }

    if (minDist[N] == INT_MAX)
        cout << -1 << endl;
    else
        cout << minDist[N] << endl;

    return 0;
}
```

---

## ⏱️ 复杂度分析

- 使用优先队列的 Dijkstra：
  - 时间复杂度：`O((N + M) log M)`（常见写法也记作 `O(M log M)`）
  - 空间复杂度：`O(N + M)`

---

## 🧠 总结

- Dijkstra 适用于 **非负权边** 的最短路
- 三个关键组件：
  1. `minDist`：记录当前最短路
  2. `visited`：确定最短路的点
  3. 小根堆：快速找到下一次扩展的最小距离点

---

📌 **一句话记忆**：

> Dijkstra = 每次取出当前距离最小的未确定点，做松弛更新

