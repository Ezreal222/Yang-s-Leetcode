# 1584. Min Cost to Connect All Points / 连接所有点的最小费用

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, MST, Prim, Kruskal, Union-Find · 图, 最小生成树, Prim, Kruskal, 并查集
    - **Link**: [LeetCode](https://leetcode.com/problems/min-cost-to-connect-all-points/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给二维平面上 `n` 个点. 任意两点之间的连接代价 = **曼哈顿距离** `|x1-x2| + |y1-y2|`. 求**连接所有点** (使任意两点连通) 的**最小总代价**.

**中文**: 连接所有点的最小总代价, 边权是曼哈顿距离.

## Key Insights

1. **🔑 问题本质: 最小生成树 (MST) / This is MST**

    "连接所有点的最小总代价" = 选 `n - 1` 条边让所有点连通且边权和最小 = **最小生成树 (Minimum Spanning Tree)**.

    > **看到"连通所有 + 最小代价"** → 立刻反应 MST. 两大算法: Prim's (生长式) 和 Kruskal's (排序式).

2. **🔑 两大 MST 算法对比 / Prim's vs Kruskal's**

    | | **Prim's** (Yang 用) | **Kruskal's** |
    |---|---|---|
    | 思路 | **从一点出发, 每次加最近的外部点** | 所有边排序, 贪心加边 (用 UF 判环) |
    | 复杂度 | O(V²) (朴素) / O(E log V) (堆) | O(E log E) |
    | 适合 | **稠密图** (V² ≈ E) | 稀疏图 |
    | 本题 | n ≤ 1000, V² = 10⁶ ✓ | 也行 |

    > 本题完全图 (E = V²/2 dense) → Prim's O(V²) 最优.

3. **🔑 Prim's 三个数组 / Prim's data structures**

    - **`minDist[v]`**: v 到当前 MST 的最小边权 (初始 INF, `minDist[0] = 0` 当起点).
    - **`inMST[v]`**: v 是否已加入 MST.
    - **`total`**: 累积代价.

    **主循环 n 次**:

    1. **挑** 未访问的最小 `minDist[u]` 的 u → 加入 MST, `total += minDist[u]`.
    2. **松弛** (relax): 对每个未访问的 v, 用 u→v 的边权更新 `minDist[v] = min(minDist[v], dist(u, v))`.

    > 关键: `minDist` 永远表示"到 MST 的最短" 而不是"到起点". 加入新节点后所有 minDist 重算.

4. **`minDist[0] = 0` 让起点首轮被选 / Seed start node**

    第 1 次主循环时, 所有节点 `minDist = INF` 除了 0 = 0 → 必选 0 进 MST. 之后从 0 出发松弛邻居.

    > 起点选谁都行 (MST 唯一 / 至少代价相同), 0 最方便.

5. **复杂度 O(n²) / Dense graph cost**

    主循环 n 次, 每次内层 "挑最小" O(n) + 松弛 O(n) → O(n²). 对 n ≤ 1000 完全够.

    > 若边稀疏可上**优先队列 Prim's** 降到 O(E log V), 但本题完全图反而不划算.

6. **替代解法: Kruskal 用 UF / Kruskal alternative**

    生成所有 `n²/2` 条边, 按边权排序, 一条一条加, **遇到不成环的就加** (UF `unite` 返回 true). 加够 `n-1` 条就停. 跟 [0684](../0684-redundant-connection/README.md) 的 UF 套路完全一致, 只是这里"先排序后加".

    > Kruskal + UF 是 MST 的另一经典实现, 见 [0684 相关题目](../0684-redundant-connection/README.md). 本题 Prim's 更紧凑.

## Solution

=== "C++"
    === "v1 推荐: Prim's O(n²) (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int minCostConnectPoints(vector<vector<int>>& points) {
                int n = points.size();
                vector<int> minDist(n, INT_MAX);
                vector<bool> inMST(n, false);
                minDist[0] = 0;                                    // 让起点首轮被选
                int total = 0;
                for (int i = 0; i < n; i++) {
                    // 挑当前 minDist 最小的未访问点
                    int u = -1;
                    for (int j = 0; j < n; j++) {
                        if (!inMST[j] && (u == -1 || minDist[j] < minDist[u])) u = j;
                    }
                    inMST[u] = true;
                    total += minDist[u];

                    // 松弛: 用 u 更新所有未访问点的 minDist
                    for (int v = 0; v < n; v++) {
                        if (!inMST[v]) {
                            int d = abs(points[v][0] - points[u][0])
                                  + abs(points[v][1] - points[u][1]);
                            if (d < minDist[v]) minDist[v] = d;
                        }
                    }
                }
                return total;
            }
        };
        ```

    === "v2: Kruskal + UF"
        ```cpp
        // UF 跟 0684 / 1971 一字不差, 略
        class UnionFind {
            vector<int> parent, rank_;
        public:
            UnionFind(int n) : parent(n), rank_(n) { iota(parent.begin(), parent.end(), 0); }
            int find(int x) { if (parent[x] != x) parent[x] = find(parent[x]); return parent[x]; }
            bool unite(int x, int y) {
                int rx = find(x), ry = find(y);
                if (rx == ry) return false;
                if (rank_[rx] < rank_[ry]) swap(rx, ry);
                parent[ry] = rx;
                if (rank_[rx] == rank_[ry]) rank_[rx]++;
                return true;
            }
        };

        class Solution {
        public:
            int minCostConnectPoints(vector<vector<int>>& points) {
                int n = points.size();
                vector<tuple<int, int, int>> edges;                // (cost, u, v)
                // ⚠ 内层从 i+1 开始, 避免自环 (i==j) 和重复边 (i,j) vs (j,i)
                for (int i = 0; i < n; i++) {
                    for (int j = i + 1; j < n; j++) {
                        int cost = abs(points[i][0] - points[j][0])
                                 + abs(points[i][1] - points[j][1]);
                        edges.emplace_back(cost, i, j);
                    }
                }
                sort(edges.begin(), edges.end());                  // 边权升序

                UnionFind uf(n);
                int total = 0, count = 0;
                for (auto& [cost, u, v] : edges) {
                    if (uf.unite(u, v)) {                          // 不成环就加
                        total += cost;
                        if (++count == n - 1) break;               // 选满 n-1 条停
                    }
                }
                return total;
            }
        };
        ```

        > **⚠ 小坑**: 内层若写 `for (int j = 1; j < n; j++)` (固定 1 起步) — 会生成自环 (i=j 时) 和重复边 ((i,j) 跟 (j,i) 都进 edges). UF 的 `unite` 对自环 / 重复边都返 false, **正确性不受影响**, 但会处理 O(n²) 多余边. 用 `j = i + 1` 是 idiomatic 写法.

=== "Python"
    ```python
    class Solution:
        def minCostConnectPoints(self, points: list[list[int]]) -> int:
            n = len(points)
            INF = float('inf')
            min_dist = [INF] * n
            in_mst = [False] * n
            min_dist[0] = 0
            total = 0

            for _ in range(n):
                # 挑 min_dist 最小的未访问点 — 一行 Pythonic
                u = min((j for j in range(n) if not in_mst[j]), key=lambda j: min_dist[j])
                in_mst[u] = True
                total += min_dist[u]
                # 松弛
                for v in range(n):
                    if not in_mst[v]:
                        d = abs(points[v][0] - points[u][0]) + abs(points[v][1] - points[u][1])
                        if d < min_dist[v]:
                            min_dist[v] = d
            return total
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} points
     * @return {number}
     */
    var minCostConnectPoints = function(points) {
        const n = points.length;
        const minDist = new Array(n).fill(Infinity);
        const inMST = new Array(n).fill(false);
        minDist[0] = 0;
        let total = 0;

        for (let i = 0; i < n; i++) {
            let u = -1;
            for (let j = 0; j < n; j++) {
                if (!inMST[j] && (u === -1 || minDist[j] < minDist[u])) u = j;
            }
            inMST[u] = true;
            total += minDist[u];
            for (let v = 0; v < n; v++) {
                if (!inMST[v]) {
                    const d = Math.abs(points[v][0] - points[u][0])
                            + Math.abs(points[v][1] - points[u][1]);
                    if (d < minDist[v]) minDist[v] = d;
                }
            }
        }
        return total;
    };
    ```

## Complexity

- **Time**: O(n²) — n 次主循环 × O(n) 内层.
- **Space**: O(n) — `minDist` + `inMST`.

## 相关题目

- [0684. Redundant Connection](../0684-redundant-connection/README.md) — UF 找成环边, Kruskal 同思想
- [1971. Find if Path Exists in Graph](../1971-find-if-path-exists-in-graph/README.md) — UF 判连通
- 1135\. Connecting Cities With Minimum Cost (待补) — 同款 MST, 给的就是图
- 1168\. Optimize Water Distribution in a Village (待补) — MST + 虚拟超级源点
- 1489\. Find Critical and Pseudo-Critical Edges in MST (待补) — MST 上找关键边
- [0743. Network Delay Time](../0743-network-delay-time/README.md) — Dijkstra 单源最短路径, 跟 Prim's 同款"挑最小 + 松弛"
- 0787\. Cheapest Flights Within K Stops (待补) — 带步数限制的最短路
