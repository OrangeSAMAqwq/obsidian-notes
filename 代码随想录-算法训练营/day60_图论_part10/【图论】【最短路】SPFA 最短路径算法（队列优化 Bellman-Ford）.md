# 📘 算法模板：SPFA 最短路径算法（队列优化 Bellman-Ford）

---

## 🧠 适用场景

SPFA（Shortest Path Faster Algorithm）是 Bellman-Ford 的队列优化版本，适用于：

- 图中包含负权边（但不能有负环）
- 需要从 **单源点** 出发，求所有点的最短路径
- 稠密图比 Dijkstra 更优，稀疏图可能性能稍差于堆优化 Dijkstra

---

## 🔧 核心思想

- 每个点在松弛时才入队
- 同一个点如果已经在队列中就不再重复加入（用 `isInQueue` 标记）
- 使用邻接表节省空间和加快遍历速度

---

## 🧱 数据结构设计

- `grid[i]`：表示从 `i` 出发的所有边（邻接表）
- `minDist[i]`：从起点出发到达点 `i` 的最短路径长度
- `queue<int>`：BFS 队列，用于处理每轮可能松弛的点
- `isInQueue[i]`：标记点是否在队列中，避免重复加入

---

## ✅ 模板代码

`````cpp
#include <iostream>
#include <vector>
#include <queue>
#include <list>
#include <climits>
using namespace std;

struct Edge {
    int to;      // 边的终点
    int val;     // 边的权重
    Edge(int t, int w): to(t), val(w) {}
};

int main() {
    int n, m, p1, p2, val;
    cin >> n >> m;

    vector<list<Edge>> grid(n + 1);
    vector<bool> isInQueue(n + 1, false); // 标记是否在队列中

    for(int i = 0; i < m; i++){
        cin >> p1 >> p2 >> val;
        grid[p1].push_back(Edge(p2, val)); // 单向边
    }

    int start = 1;
    int end = n;

    vector<int> minDist(n + 1, INT_MAX);
    minDist[start] = 0;

    queue<int> que;
    que.push(start);
    isInQueue[start] = true;

    while (!que.empty()) {
        int node = que.front(); que.pop();
        isInQueue[node] = false;

        for (Edge edge : grid[node]) {
            int to = edge.to;
            int weight = edge.val;
            if (minDist[to] > minDist[node] + weight) {
                minDist[to] = minDist[node] + weight;
                if (!isInQueue[to]) {
                    que.push(to);
                    isInQueue[to] = true;
                }
            }
        }
    }

    if (minDist[end] == INT_MAX) 
        cout << "unconnected" << endl;
    else 
        cout << minDist[end] << endl;
}
`````

---

## 🧪 输入输出样例

**输入：**

```
5 6
1 2 2
1 3 4
2 3 1
2 4 7
3 5 3
4 5 1
```

**输出：**

```
8
```

---

## 📌 注意事项

- 如果存在负权环，需要额外加一个计数器数组 `count[i]`，当某个点被松弛次数超过 `n` 次，说明图中存在负环。
- 本模板适用于 **有向图**，如果是无向图请加入反向边。

---

## 📚 相关知识点

- Bellman-Ford 算法
- 队列优化思想（即只在松弛成功时入队）
- 邻接表的使用与遍历效率

---

## 🏁 总结

> SPFA 是一种实用、容易实现的最短路径算法。  
> 虽然最坏复杂度仍为 O(nm)，但在实际工程中表现良好，是 OI、ICPC、工程竞赛中的常用利器之一。

