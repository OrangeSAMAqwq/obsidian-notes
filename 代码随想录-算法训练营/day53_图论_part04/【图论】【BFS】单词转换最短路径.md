# 🔄 单词转换最短路径（图论 + BFS）🧩

---

## 🧠 题目描述

给定一个初始字符串和目标字符串，以及若干个「中间字符串节点」，每次转换只能变更一个字符，且只能变为中间节点中存在的字符串。  

目标是判断从起点变换到终点所需的最少步数，若无法转换，则输出 `0`。

---

## 💡 解题思路：图建模 + BFS 最短路径

### 1. 建图思路

- 所有字符串（起点、终点、中间节点）看作图中的节点
- 如果两个字符串只有一个字符不同，则连接一条无权边
- 构建无向图（即两者可互换）

### 2. 路径搜索

- 使用 **BFS** 搜索最短路径
- 起点为字符串 `start` 的索引 `0`，终点为字符串 `end` 的索引 `1`
- BFS 中记录访问层数（转换次数）

---

## ✅ 关键函数解析

### `strToGrid(...)`

判断两个字符串是否只有一个字符不同，如果是，则在邻接矩阵中打通边：

```cpp
void strToGrid(vector<vector<bool>> &grid, string a, string b, int i, int j){
    int diff = 0;
    for (int k = 0; k < a.size(); k++){
        if (a[k] != b[k]) diff++;
    }
    if (diff == 1){
        grid[i][j] = true;
        grid[j][i] = true;
    }
}
```

---

## 💻 完整 C++ 代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <queue>
#include <unordered_set>
#include <string>
using namespace std;
 
// 判断是否能一步转换
void strToGrid(vector<vector<bool>> &grid, string beginStr, string endStr, int i, int j){
    int flag = 0;
    for (int k = 0; k < beginStr.size(); k++){
        if (beginStr[k] != endStr[k])
            flag++;
    }
    if (flag == 1){
        grid[i][j] = true;
        grid[j][i] = true;
    }
}
 
int main(){
    int n;
    scanf("%d", &n);

    vector<string> strS(2 + n);  // 包含起点、终点和中间词
    vector<vector<bool>> grid(2 + n, vector<bool>(2 + n, false));

    cin >> strS[0] >> strS[1];  // 起点和终点

    if (strS[0] == strS[1]){
        printf("0");
        return 0;
    }

    // 构建图（邻接矩阵）
    for (int i = 2; i < n + 2; i++){
        cin >> strS[i];
        for (int j = 0; j < i; j++){
            strToGrid(grid, strS[i], strS[j], i, j);
        }
    }

    vector<bool> visited(2 + n, false);
    queue<int> q;
    q.push(0);
    visited[0] = true;

    int ans = 0;

    while(!q.empty()){
        ans++;
        int levelSize = q.size();
        for (int i = 0; i < levelSize; i++){
            int curr = q.front();
            q.pop();
            for (int j = 0; j < grid.size(); j++){
                if (grid[curr][j] && !visited[j]){
                    if (j == 1){
                        printf("%d", ans + 1);
                        return 0;
                    }
                    visited[j] = true;
                    q.push(j);
                }
            }
        }
    }

    // 无法转换
    printf("0");
    return 0;
}
```

---

## 📊 时间复杂度分析

- 建图阶段：O(n² × m)，n 为节点数，m 为字符串长度（比较所有字符串对）
- BFS 最短路径：O(n²)

---

## 🧠 总结

| 步骤 | 技术 |
|------|------|
| 构建图 | 邻接矩阵 / 邻接表 |
| 判断邻接 | 字符串差异为 1 |
| 路径搜索 | BFS 层序遍历，记录步数 |

这是一道典型的将「字符串」抽象为「图」的转换问题，适合作为模板积累 ✅

