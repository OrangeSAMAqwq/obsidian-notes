# 🏗 Kruskal算法模板：最小生成树 MST（并查集 + 堆优化）

---

## 📌 题目背景

给定一个无向加权图，包含 `V` 个点和 `E` 条边。  
现在要求构建一棵最小生成树（MST），使得：

- 所有点都连通
- 总权值最小
- 不形成环

这是经典的图论问题，常用的两种解法为：

- Prim 算法（适合稠密图）
- Kruskal 算法（适合稀疏图）✅

---

## 📚 Kruskal 核心思想

1. 将所有边按权重从小到大排序
2. 依次加入当前最小的边
3. 使用 **并查集** 判断是否会成环
4. 若两点不在同一集合 → 合并集合、加入边

---

## 🧩 并查集函数

```cpp
int find(int u, auto &father){
    if (father[u] == u)
        return u;
    return father[u] = find(father[u], father); // 路径压缩
}

bool isSame(int u, int v, auto &father){
    return find(u, father) == find(v, father);
}

void joint(int u, int v, auto &father){
    u = find(u, father);
    v = find(v, father);
    if (u != v)
        father[u] = v;
}
```

---

## ⚙️ 自定义小根堆结构体（按权重排序）

```cpp
struct cmp{
    bool operator()(vector<int> &a, vector<int> &b){
        return a[2] > b[2]; // 小根堆，权值小的优先
    }
};
```

---

## 💻 完整代码实现（C++）

```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

struct cmp{
    bool operator()(vector<int> &a, vector<int> &b){
        return a[2] > b[2];
    }
};

int find(int u, auto &father){
    if (father[u] == u)
        return u;
    return father[u] = find(father[u], father);
}

bool isSame(int u, int v, auto &father){
    return (find(u, father) == find(v, father));
}

void joint(int u, int v, auto &father){
    u = find(u, father);
    v = find(v, father);
    if (u == v)
        return;
    father[u] = v;
}

int main(){
    std::priority_queue<vector<int>, vector<vector<int>>, cmp> pq;
    int V, E;
    cin >> V >> E;

    vector<int> father(V + 1);
    for (int i = 1; i <= V; i++){
        father[i] = i;
    }

    for (int i = 0; i < E; i++){
        int v1, v2, val;
        cin >> v1 >> v2 >> val;
        pq.push({v1, v2, val});
    }

    int ans = 0;
    while (!pq.empty()){
        vector<int> temp = pq.top(); pq.pop();
        int n1 = temp[0], n2 = temp[1], cost = temp[2];
        if (isSame(n1, n2, father)) continue;
        joint(n1, n2, father);
        ans += cost;
    }

    cout << ans << endl;
    return 0;
}
```

---

## 🧠 时间复杂度分析

- 并查集操作复杂度：`O(α(n))`，可视为常数
- 边排序使用堆：`O(E log E)`
- 总体复杂度：`O(E log E)`，适合 **稀疏图**

---

## 🧪 示例输入输出

```text
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

说明：选中的边是 (1,2)=1, (2,3)=2, (2,4)=3，权值总和为 6。

---

## 📦 应用场景

- 电网/网络/城市建造成本最小化
- 地图连通规划
- 聚类算法（图聚类）

---

## ✅ 总结

Kruskal 算法通过排序 + 并查集，有效地避免了图中形成环，  
并保证生成树的权值总和最小。适合大多数稀疏图的 MST 构建问题，是必学模板。

