# KC-97. 小明逛公园 / Floyd-Warshall All-Pairs Shortest Path

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Floyd-Warshall, All-Pairs Shortest Path, DP · 图, Floyd-Warshall, 全源最短路, 动态规划
    - **Link**: [KamaCoder 97](https://kamacoder.com/problempage.php?pid=1155) · 代码随想录
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

> KamaCoder / 代码随想录 题. **Floyd-Warshall 全源最短路** 入门 demo.

## Problem

**EN**: 公园有 n 个景点, m 条无向带权路径. 之后 z 次询问, 每次给 `start` 和 `end`, 求两点最短路 (不连通输 `-1`).

**中文**: 无向带权图 + 多次"两点最短路" 查询. Floyd-Warshall 完美场景.

## Key Insights

1. **🔑 多次查询 + 任意两点 → Floyd-Warshall / Many queries → all-pairs**

    跟 [Dijkstra (0743)](../0743-network-delay-time/README.md) / [BF (KC-94)](../kc-94-city-cargo-transport-i/README.md) 的"单源最短路" 不同, Floyd 一次求出**所有 (i, j) 对** 的最短路, 之后 O(1) 查询. 适合**多次查询小图**.

    > 单源用 Dijkstra/BF, **多源 / 多查询** 用 Floyd. **询问次数 z 多就更划算**.

2. **🔑 状态: `dist[i][j] = i 到 j 的最短路径长度` / All-pairs distance matrix**

    用二维矩阵存. 初始: 直接边的权重, 没有边的 `INF`. `dist[i][i] = 0`.

    Yang 用 `10005` 当哨兵 (足够大但不溢出). LC 范围常用 INT_MAX / 2 防 INT_MAX + INT_MAX 溢出. 但需要避免 `dist[i][k] + dist[k][j]` 溢出.

3. **🔑 转移: 枚举中转点 k / Transition: enumerate intermediate**

    $$dist[i][j] = \min(dist[i][j],\ dist[i][k] + dist[k][j])$$

    意义: i 到 j 要么不经 k (`dist[i][j]` 旧值), 要么经 k 一次 (`dist[i][k] + dist[k][j]`). 取较小.

    > 这是 DP 中的"区间 + 中转" 思想, 跟 [0312 戳气球](../../10-dp/0312-burst-balloons/README.md) "枚举最后操作" 的精神类似 — 但这里枚举的是**中转节点**.

4. **🔑 循环顺序: k 必须在最外 / k MUST be outermost loop**

    ```cpp
    for (int k = 1; k <= n; k++)                                 // ⚠ k 必须最外
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
    ```

    **为什么?** Floyd 本质是三维 DP: `dp[k][i][j] = 用编号 ≤ k 的中转点的最短路`. 转移用 `dp[k-1][...]`. 空间压缩成 2D 后, **k 在最外保证内层 i, j 算时, `dist[i][k]` 和 `dist[k][j]` 仍是上一轮 (k-1) 的值** (因为 k 自己作为中转, 不会用本轮新算的 dist).

    > 把 k 写在中间或最内 — 仍然能算出结果但**会用到本轮新算的值**, 含义就不是"恰好用前 k 个中转" 了, 可能错过最优.

5. **跟单源算法对照 / vs Single-source**

    | 算法 | 范围 | 边权 | 复杂度 |
    |---|---|---|---|
    | **Dijkstra** | 单源 | 非负 | O(V² + E log V) |
    | **Bellman-Ford** | 单源 | 任意 (可负) | O(V × E) |
    | **Floyd-Warshall** (本题) | **全源** (所有 pair) | **任意, 但无负环** | **O(V³)** |

    > Floyd 在小图 + 多查询时 O(V³) 一次性算完比"每次 BF / Dijkstra" 总成本低得多.

6. **无向图: 边对称存 / Symmetric for undirected**

    Yang 写 `dist[p1][p2] = dist[p2][p1] = val` — 无向图边对称. 漏写一条就漏一半的连接关系.

7. **复杂度 O(V³) + O(Z) / Cubic + queries**

    建表 O(V³), 每次查询 O(1) (直接读 `dist[s][t]`). 共 O(V³ + Z).

## Solution

=== "C++"
    ```cpp
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        int n, m, p1, p2, val;
        cin >> n >> m;
        // 10005 当 INF 哨兵 (足够大且不溢出: 10005 + 10005 < INT_MAX)
        vector<vector<int>> dist(n + 1, vector<int>(n + 1, 10005));
        for (int i = 1; i <= n; i++) dist[i][i] = 0;                // 自己到自己 0

        while (m--) {
            cin >> p1 >> p2 >> val;
            dist[p1][p2] = val;
            dist[p2][p1] = val;                                     // 无向图对称
        }

        // ⚠ k 必须最外层
        for (int k = 1; k <= n; k++) {
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= n; j++) {
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                }
            }
        }

        int z, start, end_;
        cin >> z;
        while (z--) {
            cin >> start >> end_;
            cout << (dist[start][end_] == 10005 ? -1 : dist[start][end_]) << endl;
        }
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
        INF = 10005
        dist = [[INF] * (n + 1) for _ in range(n + 1)]
        for i in range(n + 1):
            dist[i][i] = 0

        for _ in range(m):
            u, v, w = int(data[idx]), int(data[idx + 1]), int(data[idx + 2])
            idx += 3
            dist[u][v] = w
            dist[v][u] = w                                          # 无向图

        # k 最外: 三层循环
        for k in range(1, n + 1):
            for i in range(1, n + 1):
                for j in range(1, n + 1):
                    if dist[i][k] + dist[k][j] < dist[i][j]:
                        dist[i][j] = dist[i][k] + dist[k][j]

        z = int(data[idx]); idx += 1
        out = []
        for _ in range(z):
            s, t = int(data[idx]), int(data[idx + 1]); idx += 2
            out.append(str(-1 if dist[s][t] == INF else dist[s][t]))
        print("\n".join(out))

    main()
    ```

=== "JavaScript"
    ```javascript
    const data = require('fs').readFileSync(0, 'utf8').split(/\s+/).map(Number);
    let idx = 0;
    const n = data[idx++], m = data[idx++];
    const INF = 10005;
    const dist = Array.from({length: n + 1}, () => new Array(n + 1).fill(INF));
    for (let i = 0; i <= n; i++) dist[i][i] = 0;

    for (let i = 0; i < m; i++) {
        const u = data[idx++], v = data[idx++], w = data[idx++];
        dist[u][v] = w;
        dist[v][u] = w;
    }

    for (let k = 1; k <= n; k++) {
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }

    const z = data[idx++];
    const out = [];
    for (let i = 0; i < z; i++) {
        const s = data[idx++], t = data[idx++];
        out.push(dist[s][t] === INF ? -1 : dist[s][t]);
    }
    console.log(out.join("\n"));
    ```

## Complexity

- **Time**: O(V³) 建表 + O(Z) 查询 = O(V³ + Z).
- **Space**: O(V²) — 矩阵.

## 相关题目

- [0743. Network Delay Time](../0743-network-delay-time/README.md) — Dijkstra 单源最短路
- [KC-94. 城市间货物运输 I](../kc-94-city-cargo-transport-i/README.md) — BF 单源最短路
- [KC-95. 城市间货物运输 II](../kc-95-city-cargo-transport-ii/README.md) — BF + 负环判定
- [0787. Cheapest Flights Within K Stops](../0787-cheapest-flights-within-k-stops/README.md) — BF + 步数限制
- 1334\. Find the City With the Smallest Number of Neighbors at a Threshold Distance (待补) — **LC 版 Floyd** 经典应用
- 0399\. Evaluate Division (待补) — 带权图 + DFS / 并查集 / Floyd 都行
- 2642\. Design Graph With Shortest Path Calculator (待补) — 设计题, 适合 Floyd
