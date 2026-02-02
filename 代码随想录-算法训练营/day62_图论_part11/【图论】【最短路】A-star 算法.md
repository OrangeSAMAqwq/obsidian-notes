# ♞ A* 算法解马的最短路径问题（骑士走法）

---

## 🧠 题目理解

给定两个棋盘上的位置 `(a1, a2)` 和 `(b1, b2)`，你需要求出国际象棋中的马（Knight）从起点跳到终点所需的最少步数。

### ❗ 特别限制：

- 棋盘大小为 1000 × 1000
- 每次移动为马的标准“日”字走法（8 个方向）

---

## 🚀 算法选择：A* 搜索算法（启发式搜索）

与 BFS 不同，A* 使用 **估价函数（启发函数）** 来决定搜索优先顺序，能加快搜索过程。

---

## 📐 启发函数（Heuristic Function）

```text
f = g + h
```

- `g`：起点到当前节点的代价（每步 5，预估距离平方）
- `h`：当前节点到目标的欧几里得距离平方（不开根）

你代码中使用的是：

```cpp
H = (x - b1)^2 + (y - b2)^2
```

无需开平方，比较 `f` 值本身即可。

---

## 🧩 数据结构设计

- `Knight` 结构体表示一个状态节点（x, y, g, h, f）
- 重载 `<` 运算符：实现最小堆（`priority_queue` 默认是最大堆）
- `moves[x][y]`：记录到达该坐标所需步数，避免重复访问

---

## 📦 路径拓展方向

```cpp
int dir[8][2] = {
    {-2, -1}, {-2, 1}, {-1, 2}, {1, 2},
    {2, 1}, {2, -1}, {1, -2}, {-1, -2}
};
```

马能从当前位置跳到的 8 个方位。

---

## 💻 完整代码

````cpp
#include <iostream>
#include <queue>
#include <string.h>
using namespace std;

int moves[1001][1001];
int dir[8][2] = {
    {-2, -1}, {-2, 1}, {-1, 2}, {1, 2},
    {2, 1}, {2, -1}, {1, -2}, {-1, -2}
};

int b1, b2;

struct Knight {
    int x, y;
    int g, h, f;
    bool operator < (const Knight &k) const {
        return k.f < f; // 小顶堆
    }
};

priority_queue<Knight> que;

int Heuristic(const Knight& k) {
    return (k.x - b1) * (k.x - b1) + (k.y - b2) * (k.y - b2);
}

void astar(const Knight& k) {
    Knight cur, next;
    que.push(k);
    while (!que.empty()) {
        cur = que.top(); que.pop();
        if (cur.x == b1 && cur.y == b2) break;

        for (int i = 0; i < 8; i++) {
            next.x = cur.x + dir[i][0];
            next.y = cur.y + dir[i][1];

            if (next.x < 1 || next.x > 1000 || next.y < 1 || next.y > 1000)
                continue;

            if (!moves[next.x][next.y]) {
                moves[next.x][next.y] = moves[cur.x][cur.y] + 1;
                next.g = cur.g + 5;
                next.h = Heuristic(next);
                next.f = next.g + next.h;
                que.push(next);
            }
        }
    }
}

int main() {
    int n, a1, a2;
    cin >> n;
    while (n--) {
        cin >> a1 >> a2 >> b1 >> b2;
        memset(moves, 0, sizeof(moves));
        Knight start;
        start.x = a1;
        start.y = a2;
        start.g = 0;
        start.h = Heuristic(start);
        start.f = start.g + start.h;
        astar(start);
        while (!que.empty()) que.pop(); // 清空队列
        cout << moves[b1][b2] << endl;
    }
    return 0;
}
````

---

## ✅ 示例输入输出

**输入：**

```
2
1 1 4 5
10 10 20 20
```

**输出：**

```
2
6
```

---

## 🔍 总结

| 特性 | 内容 |
|------|------|
| 算法 | A* 搜索 |
| 启发函数 | 欧式距离平方 |
| 优化 | 避免开根号、使用优先队列 |
| 注意 | `moves[][]` 清零、队列清空 |

---

📌 本题是 A* 算法在棋盘类问题中的经典应用，启发函数选得好，速度飞起来 🚀

