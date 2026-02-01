# 📘 算法模板：Bellman-Ford 最短路径算法（含负环判断）

---

## 🧠 适用场景

Bellman-Ford 是一种适用于 **带负权边** 的最短路径算法，支持以下情况：

- **边权可为负数**
- 可检测 **负权环**
- 不适合高性能要求（效率低于 Dijkstra）

---

## 🔧 核心思想

1. 从起点开始，最多进行 `n-1` 轮松弛
2. 每轮尝试用某条边更新目标点的最短距离
3. 第 `n` 轮用于判断是否存在负环（如果还能更新，说明存在负权环）

---

## 📚 数据结构设计

- `grid`：边列表，每条边由 `{from, to, weight}` 三元组构成
- `minDist[i]`：表示从起点出发，到达第 `i` 个点的最短路径
- `flag`：用于记录是否存在负权环

---

## ✅ 模板代码

`````cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

int main() {
    int n, m, p1, p2, val;
    cin >> n >> m;

    vector<vector<int>> grid;

    for(int i = 0; i < m; i++){
        cin >> p1 >> p2 >> val;
        grid.push_back({p1, p2, val});
    }

    int start = 1;  // 起点
    int end = n;    // 终点

    vector<int> minDist(n + 1, INT_MAX);
    minDist[start] = 0;
    bool flag = false;

    for (int i = 1; i <= n; i++) { // 共执行 n 次
        for (vector<int> &side : grid) {
            int from = side[0];
            int to = side[1];
            int price = side[2];
            if (minDist[from] != INT_MAX && minDist[to] > minDist[from] + price) {
                if (i == n) {
                    flag = true; // 第 n 次还能更新 → 存在负权环
                } else {
                    minDist[to] = minDist[from] + price;
                }
            }
        }
    }

    if (flag) cout << "circle" << endl;           // 存在负环
    else if (minDist[end] == INT_MAX) 
        cout << "unconnected" << endl;            // 无法到达终点
    else 
        cout << minDist[end] << endl;             // 输出最短路径
}
`````

---

## 📌 注意事项

- 时间复杂度：`O(n * m)`，其中 `n` 是节点数，`m` 是边数
- 空间复杂度：`O(n)`
- 若仅求最短路径，无需执行第 `n` 次循环
- 本模板适用于 **单源最短路径**

---

## 🧪 输入输出样例

**输入：**

```
4 5
1 2 3
1 3 8
2 3 -4
2 4 5
3 4 2
```

**输出：**

```
1
```

---

## 🔁 与其它算法对比

| 算法 | 支持负边 | 可检测负环 | 时间复杂度 | 适用场景 |
|------|-----------|-------------|-------------|-----------|
| Dijkstra（堆优化） | ❌ | ❌ | O(mlog n) | 正权图 |
| Bellman-Ford | ✅ | ✅ | O(nm) | 含负边图 |
| SPFA（Bellman-Ford 队列优化） | ✅ | ✅ | 期望 O(m) | 工程实践 |

---

## 🏁 总结

> Bellman-Ford 是解决 **负权图最短路径问题的标准工具**，在图中有负边或需检测负环时是首选方案。

