# 题目：可达路径（DFS + 回溯）🌉🧠

## 🧩 题意理解

题目可以抽象为一个图搜索问题：

- 给定一张 **有向图**
- 起点固定为 `1`
- 终点固定为 `N`
- 需要找出 **所有从 1 到 N 的可达路径**
- 在一条路径中，节点 **不能重复出现**

如果不存在任何从 1 到 N 的路径，则结果为 `-1`。

---

## 💡 核心思想

这是一个典型的 **图中所有路径枚举问题**。

整体思路为：

- 使用 **DFS** 不断向前探索路径
- 使用 **回溯** 在递归返回时撤销选择
- 使用 `visited` 数组防止在一条路径中出现环

---

## 🔍 算法设计

### ① 图的表示

使用邻接矩阵表示图结构：

```text
grid[a][b] = true
```

表示存在一条从节点 `a` 指向节点 `b` 的有向边。

由于节点数量较小，邻接矩阵实现简单直观。

---

### ② DFS 搜索逻辑

DFS 过程中需要维护以下状态：

```text
cur      当前所在节点
path     当前路径
visited  节点访问标记数组
ans      所有合法路径的集合
```

递归规则：

- 若 `cur == N`，说明找到一条完整路径，加入结果
- 否则枚举所有可能的下一个节点，继续 DFS

---

### ③ 回溯细节（关键）

每一次 DFS 返回之前，都必须：

- 恢复 `visited` 状态
- 弹出路径末尾节点

这是保证不同路径之间互不干扰的核心。

---

## ✅ C++ 实现代码

```cpp
#include <iostream>
#include <vector>
using namespace std;

void dfs(vector<vector<int>>& ans,
         vector<int>& path,
         const vector<vector<bool>>& grid,
         vector<bool>& visited,
         int cur,
         int N) {
    if (cur == N) {
        ans.push_back(path);
        return;
    }

    for (int nxt = 2; nxt <= N; ++nxt) {
        if (!visited[nxt] && grid[cur][nxt]) {
            visited[nxt] = true;
            path.push_back(nxt);
            dfs(ans, path, grid, visited, nxt, N);
            path.pop_back();
            visited[nxt] = false;
        }
    }
}

int main() {
    int N, M;
    cin >> N >> M;

    vector<vector<bool>> grid(N + 1, vector<bool>(N + 1, false));
    vector<bool> visited(N + 1, false);

    for (int i = 0; i < M; ++i) {
        int a, b;
        cin >> a >> b;
        grid[a][b] = true;
    }

    vector<vector<int>> ans;
    vector<int> path;
    path.push_back(1);
    visited[1] = true;

    dfs(ans, path, grid, visited, 1, N);

    if (ans.empty()) {
        cout << -1 << endl;
    } else {
        for (const auto& p : ans) {
            for (int i = 0; i < p.size(); ++i) {
                if (i) cout << ' ';
                cout << p[i];
            }
            cout << endl;
        }
    }
    return 0;
}
```

---

## 🧠 总结

- 本题是 **DFS + 回溯** 的经典应用
- 关键点在于：
  - 正确建图（有向）
  - 使用 `visited` 防止路径中出现环
  - 到达终点立即收集路径
- 该模板可以直接迁移到：
  - 图中所有路径问题
  - 小规模图搜索问题

---
