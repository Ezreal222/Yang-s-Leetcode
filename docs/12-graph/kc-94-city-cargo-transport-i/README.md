# KC-94. 城市间货物运输 I / City Cargo Transport I

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Shortest Path, Bellman-Ford · 图, 最短路, Bellman-Ford
    - **Link**: [KamaCoder 94](https://kamacoder.com/problempage.php?pid=1152) · 代码随想录
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

> **KamaCoder / 代码随想录** 题, 不是 LC. 作为 **Bellman-Ford 算法** 的入门 demo 题.

## Problem

**EN**: 有向带权图, n 个城市 m 条边 (**边权可负**). 从城市 1 出发, 求到城市 n 的**最短路**. 不可达输出 `"unconnected"`.

**中文**: 有向带权图 (可能有负权边), 求 1 → n 最短路, 不连通输出 `unconnected`.

## Key Insights

1. **🔑 边权可负 → Dijkstra 失效 → Bellman-Ford / Negative weights kill Dijkstra**

    [Dijkstra (0743)](../0743-network-delay-time/README.md) 的**贪心证明** 依赖"非负边权". 一旦有负边, 贪心选出的 `minDist[u]` **不再是最终最短** — 因为后续可能通过负边绕回来更短.

    > **看到边权可负 → 立刻反应 Bellman-Ford**. 是单源最短路的"通用解".

2. **🔑 Bellman-Ford 核心: 松弛所有边 `n - 1` 次 / Relax all edges n-1 times**

    主循环:

    ```
    for i = 1 to n - 1:
        for each edge (u, v, w):
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    ```

    > **每一轮**: 把所有边过一遍, 尝试松弛. **共 n - 1 轮**.

3. **🔑 为什么是 `n - 1` 轮? / Why n - 1 iterations**

    最短路最多经过 `n` 个顶点 → 最多 `n - 1` 条边. 第 i 轮松弛后, **所有"最短路含 ≤ i 条边" 的目标节点** 都被算出. 跑 n - 1 轮 → 所有节点的最短路都收敛.

    > 这是 BF 算法正确性的核心. 跑少了某些路径还没"传播" 到; 跑多了浪费 (但不出错).

4. **🔑 `if (dist[from] != INT_MAX)` 防溢出 / Guard against INT_MAX overflow**

    Yang 写的关键守卫: 若 `dist[from]` 还是 INT_MAX (没到达过), `dist[from] + price` 会溢出 → 错误更新 dist[to]. 必须先判 `from` 已可达.

    > **整型溢出是 BF 实现里最经典的 bug**. 加这条 if 是必修.

5. **判负环 (Yang 没做, 但模板要懂) / Negative cycle detection**

    BF 多跑**一轮** 看是否还有 dist 更新: 有 → 存在**负环** (从起点可达). 题目若需要可加:

    ```cpp
    for (auto& edge : graph) {
        if (dist[edge[0]] != INT_MAX && dist[edge[1]] > dist[edge[0]] + edge[2]) {
            // 第 n 轮还能松弛 → 有负环
            return "NEGATIVE_CYCLE";
        }
    }
    ```

    > 本题保证无负环, 没写; 但完整 BF 必带这步.

6. **跟 [Dijkstra](../0743-network-delay-time/README.md) / SPFA 对照 / Compare**

    | 算法 | 边权 | 复杂度 | 备注 |
    |---|---|---|---|
    | **Dijkstra** | **非负** | O((V + E) log V) | 贪心, 不能负边 |
    | **Bellman-Ford** | **任意 (可负)** | O(V × E) | 通用, 慢 |
    | **SPFA** | **任意 (可负)** | 平均 O(k × E), 最坏 O(V × E) | BF 的队列优化, 实测快但理论同 BF |
    | **Floyd-Warshall** | 任意 (可负, 无负环) | O(V³) | 全源最短路 |

    > **边权决定选谁**: 全非负 → Dijkstra; 有负 → BF/SPFA; 全源 → Floyd.

7. **复杂度 O(V × E) / V·E**

    主循环 V - 1 次, 每次扫所有 E 条边. 总 O(V × E).

## Solution

=== "C++"
    ```cpp
    #include <iostream>
    #include <climits>
    #include <vector>
    using namespace std;

    int main() {
        int n, m, p1, p2, val;
        cin >> n >> m;
        vector<vector<int>> graph;                                          // 边列表 (from, to, w)
        while (m--) {
            cin >> p1 >> p2 >> val;
            graph.push_back({p1, p2, val});
        }

        vector<int> dist(n + 1, INT_MAX);
        dist[1] = 0;

        // 松弛 n - 1 轮
        for (int i = 1; i < n; i++) {
            for (auto& edge : graph) {
                int from = edge[0], to = edge[1], price = edge[2];
                // ⚠ 防 INT_MAX + price 溢出 — BF 经典坑
                if (dist[from] != INT_MAX && dist[to] > dist[from] + price) {
                    dist[to] = dist[from] + price;
                }
            }
        }

        if (dist[n] == INT_MAX) cout << "unconnected";
        else cout << dist[n];
        return 0;
    }
    ```

=== "Python"
    ```python
    # KamaCoder 风格的 stdin 处理
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

        # 松弛 n - 1 轮
        for _ in range(n - 1):
            for u, v, w in edges:
                if dist[u] != INF and dist[u] + w < dist[v]:                # 同款防溢出
                    dist[v] = dist[u] + w

        print(dist[n] if dist[n] != INF else "unconnected")

    main()
    ```

=== "JavaScript"
    ```javascript
    // Node.js stdin 读取
    const lines = require('fs').readFileSync(0, 'utf8').trim().split('\n');
    const [n, m] = lines[0].split(' ').map(Number);
    const edges = [];
    for (let i = 1; i <= m; i++) {
        edges.push(lines[i].split(' ').map(Number));
    }

    const dist = new Array(n + 1).fill(Infinity);
    dist[1] = 0;

    for (let i = 1; i < n; i++) {
        for (const [u, v, w] of edges) {
            if (dist[u] !== Infinity && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }

    console.log(dist[n] === Infinity ? "unconnected" : dist[n]);
    ```

## Complexity

- **Time**: O(V × E) — 松弛 V - 1 轮, 每轮过 E 条边.
- **Space**: O(V + E) — dist + 边列表.

## 相关题目

- [0743. Network Delay Time](../0743-network-delay-time/README.md) — LC 版本的 Dijkstra (非负边权)
- [1584. Min Cost to Connect All Points](../1584-min-cost-to-connect-all-points/README.md) — Prim's MST, 同款"挑最小 + 松弛"
- [0787. Cheapest Flights Within K Stops](../0787-cheapest-flights-within-k-stops/README.md) — **BF LC 经典应用**, 加步数限制 + 快照
- 0743\. Network Delay Time — 同款单源最短路, 但非负边权用 Dijkstra
- 1334\. Find the City With the Smallest Number of Neighbors at a Threshold Distance (待补) — Floyd-Warshall 全源最短路
- [KC-95. 城市间货物运输 II](../kc-95-city-cargo-transport-ii/README.md) — 本题加**负环判定** (跑第 n 轮看是否还能松弛)
- KC-96\. 城市间货物运输 III (待补) — SPFA 优化版
