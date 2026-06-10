# 0787. Cheapest Flights Within K Stops / K 站中转内最便宜的航班

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Bellman-Ford, Shortest Path · 图, Bellman-Ford, 最短路
    - **Link**: [LeetCode](https://leetcode.com/problems/cheapest-flights-within-k-stops/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `n` 个城市, `flights[i] = [u, v, w]` 表示 u → v 的航班票价 w. 从 `src` 到 `dst`, 中转**至多 k 次** (即航段最多 `k + 1` 段). 求最少花费, 不可达返 `-1`.

**中文**: 中转 ≤ k 次 (航段 ≤ k+1), src → dst 最少票价.

## Key Insights

1. **🔑 k 次中转 ≤ k+1 条边 → BF 跑 k+1 轮 / k stops = k+1 edges = BF k+1 iterations**

    BF 跑第 `i` 轮后, 得到的是"**最多用 i 条边** 的最短路径". 题目限制中转 ≤ k 次 = 边 ≤ k+1 条 → **正好跑 k+1 轮**.

    > **"边数限制 + 最短路" 的天然 BF 场景**. Dijkstra 没有"轮数 = 边数" 的语义, 处理不了这种限制.

2. **🔑 必须用 `prev_dist` 快照 — 否则可能一轮用多条边 / Snapshot to enforce one-edge-per-round**

    Yang 的关键: 每轮开头 `prev_dist = dist`, 然后**只用 prev_dist 松弛** (不用本轮已更新的 dist):

    ```cpp
    for (int i = 1; i <= k + 1; i++) {
        prev_dist = dist;                                    // ⚠ 快照
        for (auto& f : flights) {
            if (prev_dist[u] != INT_MAX) {
                dist[v] = min(dist[v], prev_dist[u] + p);    // 用快照, 不是 dist[u]
            }
        }
    }
    ```

    **不快照会出错**: 同一轮内一边松弛 `dist[v1]`, 一边用更新后的 `dist[v1]` 松弛 `dist[v2]` → 这条路径用了 2 条边但只跑了 1 轮 → 边数超限. **快照让每轮恰好"+1 条边"**.

    > 这是 BF 处理"边数约束" 的灵魂. **没快照写不对 0787**.

3. **跟 [KC-94/95 普通 BF](../kc-94-city-cargo-transport-i/README.md) 的差异 / vs unconstrained BF**

    | | KC-94 / 95 | **0787 (本题)** |
    |---|---|---|
    | 问 | 任意条边的最短路 | **最多 k+1 条边** 的最短路 |
    | 轮数 | n - 1 (足以收敛) | **正好 k + 1** |
    | 快照 | 不需要 (无所谓多边) | **必需** (强制每轮 +1 边) |

    > 边数有限制时, BF 的"每轮 +1 边" 语义被显式利用. 普通 BF 不带快照也行因为收敛后多跑无影响; 本题不带快照会算多条边的路径, 直接 WA.

4. **`prev_dist[u] != INT_MAX` 防溢出 / Same overflow guard as BF**

    跟 [KC-94](../kc-94-city-cargo-transport-i/README.md) 同款守卫. INT_MAX + price 溢出会错误更新 dist[v].

5. **复杂度 O((k + 1) × E) / Linear in k and edges**

    主循环 k + 1 次, 每次扫所有 E 条边. 总 O(k × E).

    > LC 数据 n ≤ 100, edges ≤ 10⁴, k ≤ n → O(10⁶), 完全可以.

6. **替代解法: Dijkstra + 状态加一维 / Alternative: Dijkstra with stops dimension**

    把状态从 `(node)` 升级到 `(node, stops_used)`, Dijkstra 出最小 cost. 但要小心 stops_used 单调性 — 同 node 不同 stops 不能直接剪枝. 实现细节比 BF 多.

    > BF 版更紧凑, 推荐. Dijkstra 版适合 k 很大 / 边权很大需要堆加速时.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
            vector<int> dist(n + 1, INT_MAX);
            vector<int> prev_dist(n + 1);
            dist[src] = 0;

            // 跑 k + 1 轮 (中转 k 次 = 边数 ≤ k+1)
            for (int i = 1; i <= k + 1; i++) {
                prev_dist = dist;                              // ⚠ 必须快照
                for (auto& f : flights) {
                    int u = f[0], v = f[1], p = f[2];
                    if (prev_dist[u] != INT_MAX) {
                        dist[v] = min(dist[v], prev_dist[u] + p);  // 用快照
                    }
                }
            }

            return dist[dst] == INT_MAX ? -1 : dist[dst];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findCheapestPrice(self, n: int, flights: list[list[int]], src: int, dst: int, k: int) -> int:
            INF = float('inf')
            dist = [INF] * (n + 1)
            dist[src] = 0

            for _ in range(k + 1):
                # list[:] 是浅拷贝, 等价 C++ vector 复制赋值
                prev_dist = dist[:]
                for u, v, p in flights:
                    if prev_dist[u] != INF:
                        if prev_dist[u] + p < dist[v]:
                            dist[v] = prev_dist[u] + p
            return -1 if dist[dst] == INF else dist[dst]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @param {number[][]} flights
     * @param {number} src
     * @param {number} dst
     * @param {number} k
     * @return {number}
     */
    var findCheapestPrice = function(n, flights, src, dst, k) {
        const dist = new Array(n + 1).fill(Infinity);
        dist[src] = 0;
        for (let i = 1; i <= k + 1; i++) {
            // [...arr] spread 浅拷贝快照
            const prev = [...dist];
            for (const [u, v, p] of flights) {
                if (prev[u] !== Infinity) {
                    dist[v] = Math.min(dist[v], prev[u] + p);
                }
            }
        }
        return dist[dst] === Infinity ? -1 : dist[dst];
    };
    ```

## Complexity

- **Time**: O((k + 1) × E).
- **Space**: O(n) — dist + prev_dist.

## 相关题目

- [KC-94. 城市间货物运输 I](../kc-94-city-cargo-transport-i/README.md) — BF 母题 (无边数限制)
- [KC-95. 城市间货物运输 II](../kc-95-city-cargo-transport-ii/README.md) — BF + 负环判定
- [0743. Network Delay Time](../0743-network-delay-time/README.md) — Dijkstra 单源最短路 (非负边权)
- 0743\. (重复) Network Delay Time
- 1631\. Path With Minimum Effort (待补) — 网格 Dijkstra (边权 = 高度差)
- 0913\. Cat and Mouse (待补) — BFS 多状态博弈
- 2045\. Second Minimum Time to Reach Destination (待补) — BFS / Dijkstra 找次短路
