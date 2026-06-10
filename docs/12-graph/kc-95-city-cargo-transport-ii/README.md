# KC-95. 城市间货物运输 II / City Cargo Transport II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Shortest Path, Bellman-Ford, Negative Cycle · 图, 最短路, Bellman-Ford, 负环
    - **Link**: [KamaCoder 95](https://kamacoder.com/problempage.php?pid=1153) · 代码随想录
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

> KamaCoder / 代码随想录 题. **[KC-94](../kc-94-city-cargo-transport-i/README.md) 的进阶** — 加**负环判定**.

## Problem

**EN**: 同 [KC-94](../kc-94-city-cargo-transport-i/README.md), 但**可能有负环**. 若 1 → n 路径含负环, 输出 `"circle"`; 否则同 KC-94 (有路径输出 `dist`, 不可达输出 `"unconnected"`).

**中文**: BF 算法 + 负环判定. 有负环输 circle, 不可达输 unconnected, 否则输最短路.

## Key Insights

1. **🔑 多跑第 n 轮看是否还能松弛 / Extra n-th iteration for cycle detection**

    BF 经典定理: **跑 n - 1 轮后所有最短路已收敛**. 若**第 n 轮** 还有边能松弛 → 必存在**从起点可达的负环** (新加的负环让某条路径"无限变短", 永远松弛).

    > **判负环 = 多跑一轮看是否更新**. 这是 BF 比 Dijkstra 多出的"附加能力".

2. **🔑 实现: 把"判负环" 的最后一轮统一写在主循环里 / Unified loop with phase guard**

    Yang 巧妙地把 n 轮统一进一个循环, 用 `if (i < n)` 区分:

    ```cpp
    for (int i = 1; i <= n; i++) {                                          // 跑 n 轮 (n-1 松弛 + 1 检测)
        for (auto& edge : graph) {
            if (i < n) {
                // 前 n - 1 轮: 正常松弛
                if (dist[from] != INT_MAX && dist[to] > dist[from] + price)
                    dist[to] = dist[from] + price;
            } else {
                // 第 n 轮: 只看能否松弛, 不实际更新
                if (dist[from] != INT_MAX && dist[to] > dist[from] + price)
                    flag = true;
            }
        }
    }
    ```

    > 等价写法是"分两段": 先单独 for n-1 轮做松弛, 再单独 for 一遍判负环. Yang 这样写代码更紧凑, 思路一致.

3. **🔑 为什么"可达负环" 才算? / Why only reachable from source**

    BF 求的是"**从起点出发的最短路**". 跟起点 1 不连通的负环跟 1 → n 路径无关, 不应该报"circle". Yang 的代码自带这层过滤 — `dist[from] != INT_MAX` 守卫确保只有"已被起点到达" 的节点上的边才能触发标记.

    > 这条**关于"可达性"** 的过滤是细节关键. 漏写会把"图里有负环但跟起点没关系" 的图错判.

4. **输出优先级: 负环 > 不可达 > 最短路 / Priority of outputs**

    Yang 的最终输出顺序:

    1. `flag == true` → `"circle"` (优先级最高)
    2. `dist[n] == INT_MAX` → `"unconnected"`
    3. 否则 → 最短路值

    > **顺序不能反**: 若先判 `unconnected`, 但负环让 dist[n] 一直减小, 没被错判但语义不严谨; 而**负环的存在比"不连通" 更基本**.

5. **跟 [KC-94](../kc-94-city-cargo-transport-i/README.md) 的 diff / Minimal upgrade from KC-94**

    | | KC-94 | **KC-95 (本题)** |
    |---|---|---|
    | 轮数 | n - 1 | **n** |
    | 第 n 轮的作用 | 不跑 | **判负环** |
    | 输出 | `dist[n]` 或 `unconnected` | 加一个 `"circle"` 分支 |

    > **改三行就升级了**. 这是 BF 模板的标配, 真正写 BF 都该带这一步.

6. **复杂度 O(V × E) / Same as BF**

    多一轮 (n vs n - 1) 不改变阶. 仍 O(V × E).

## Solution

=== "C++"
    ```cpp
    #include <vector>
    #include <iostream>
    #include <climits>
    using namespace std;

    int main() {
        int n, m, p1, p2, val;
        cin >> n >> m;
        vector<vector<int>> graph;
        while (m--) {
            cin >> p1 >> p2 >> val;
            graph.push_back({p1, p2, val});
        }

        vector<int> dist(n + 1, INT_MAX);
        dist[1] = 0;
        bool flag = false;

        // 跑 n 轮: 前 n - 1 松弛, 第 n 判负环
        for (int i = 1; i <= n; i++) {
            for (auto& edge : graph) {
                int from = edge[0], to = edge[1], price = edge[2];
                if (dist[from] == INT_MAX) continue;                        // 防溢出 + 过滤"起点不可达"
                if (i < n) {
                    if (dist[to] > dist[from] + price) {
                        dist[to] = dist[from] + price;
                    }
                } else {
                    if (dist[to] > dist[from] + price) flag = true;         // 第 n 轮还能松弛 → 负环
                }
            }
        }

        if (flag) cout << "circle";
        else if (dist[n] == INT_MAX) cout << "unconnected";
        else cout << dist[n];
        return 0;
    }
    ```

=== "Python"
    ```python
    import sys

    def main():
        data = sys.stdin.read().split()
        idx = 0
        n, m = int(data[idx]), int(data[idx + 1]); idx += 2
        edges = []
        for _ in range(m):
            u, v, w = int(data[idx]), int(data[idx + 1]), int(data[idx + 2])
            idx += 3
            edges.append((u, v, w))

        INF = float('inf')
        dist = [INF] * (n + 1)
        dist[1] = 0
        flag = False

        for i in range(1, n + 1):                                           # 跑 n 轮
            for u, v, w in edges:
                if dist[u] == INF:
                    continue
                if i < n:
                    if dist[u] + w < dist[v]:
                        dist[v] = dist[u] + w
                else:
                    if dist[u] + w < dist[v]:                               # 第 n 轮还能松弛 → 负环
                        flag = True

        if flag:
            print("circle")
        elif dist[n] == INF:
            print("unconnected")
        else:
            print(dist[n])

    main()
    ```

=== "JavaScript"
    ```javascript
    const lines = require('fs').readFileSync(0, 'utf8').trim().split('\n');
    const [n, m] = lines[0].split(' ').map(Number);
    const edges = [];
    for (let i = 1; i <= m; i++) {
        edges.push(lines[i].split(' ').map(Number));
    }

    const dist = new Array(n + 1).fill(Infinity);
    dist[1] = 0;
    let flag = false;

    for (let i = 1; i <= n; i++) {
        for (const [u, v, w] of edges) {
            if (dist[u] === Infinity) continue;
            if (i < n) {
                if (dist[u] + w < dist[v]) dist[v] = dist[u] + w;
            } else {
                if (dist[u] + w < dist[v]) flag = true;
            }
        }
    }

    if (flag) console.log("circle");
    else if (dist[n] === Infinity) console.log("unconnected");
    else console.log(dist[n]);
    ```

## Complexity

- **Time**: O(V × E) — 多跑一轮, 阶不变.
- **Space**: O(V + E).

## 相关题目

- [KC-94. 城市间货物运输 I](../kc-94-city-cargo-transport-i/README.md) — 本题母题, BF 不判负环版
- [0743. Network Delay Time](../0743-network-delay-time/README.md) — Dijkstra (非负边权)
- [0787. Cheapest Flights Within K Stops](../0787-cheapest-flights-within-k-stops/README.md) — BF + 步数限制 (快照防超边)
- KC-96\. 城市间货物运输 III (待补) — SPFA 队列优化版
- 1334\. Find the City With the Smallest Number of Neighbors at a Threshold Distance (待补) — Floyd-Warshall 全源
