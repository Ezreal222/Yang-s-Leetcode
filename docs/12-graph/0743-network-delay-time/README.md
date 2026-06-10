# 0743. Network Delay Time / 网络延迟时间

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Dijkstra, Shortest Path · 图, 单源最短路, Dijkstra 算法
    - **Link**: [LeetCode](https://leetcode.com/problems/network-delay-time/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `n` 个节点的**有向带权图**. `times[i] = [u, v, w]` 表示从 u 到 v 的延迟 w. 从节点 k 同时向所有节点发送信号, 求**所有节点都收到信号需要的最短时间**. 若不可达, 返 `-1`.

**中文**: 有向带权图, k 点发信号, 求最远节点收到的时间. 不可达返 -1.

## Key Insights

1. **🔑 单源最短路 → Dijkstra / Single-source shortest path = Dijkstra**

    "k 到所有节点的延迟" = k 到每个节点的最短路径. 答案 = **所有最短路径中的最大值** (最远那个节点的到达时间).

    > **看到"带权图 + 从一点出发求最短" → Dijkstra**. BFS 只适用于无权图 / 所有边权相等的图.

2. **🔑 Dijkstra 算法骨架: 跟 [1584 Prim's MST](../1584-min-cost-to-connect-all-points/README.md) 几乎一样 / Same shape as Prim's**

    | | [Prim's MST (1584)](../1584-min-cost-to-connect-all-points/README.md) | **Dijkstra (本题)** |
    |---|---|---|
    | `minDist[v]` | v 到**当前 MST 集合** 的最短**边** | v 到**起点 k** 的最短**路径** |
    | 松弛公式 | `min(minDist[v], weight(u, v))` | **`min(minDist[v], minDist[u] + weight(u, v))`** |
    | 答案 | 累加 minDist 之和 | 取 max(minDist) |
    | 通用框架 | "挑最小 + 松弛" | "挑最小 + 松弛" (同款) |

    > **Dijkstra ≈ Prim's 但累加 `minDist[u] +` 而非直接取边权**. 这是两个算法最易混淆的地方.

3. **🔑 三件套: `minDist` + `visited` + `graph` / Three structures**

    - **`minDist[v]`**: k 到 v 的当前最短距离 (初始 INF, `minDist[k] = 0`).
    - **`visited[v]`**: v 是否已"确定"最短距离 (出 PQ / 选出后).
    - **`graph[u][v]`**: 边权 (邻接矩阵存稠密图).

    **主循环 n 次**:

    1. 挑未确定的 `minDist[u]` 最小的 u → `visited[u] = true`.
    2. 松弛: 对每个未确定 v, 若 `minDist[u] + weight(u, v) < minDist[v]`, 更新.

4. **🔑 为什么松弛后的 u 一定是最短? / Why Dijkstra works (贪心证明)**

    每次挑当前 `minDist` 最小的未确定节点 u, 这个 `minDist[u]` 就是 k 到 u 的真正最短.

    **理由**: 任何"未来路径" 到 u 都必须经过某个更"远"(未确定的)的节点, 即至少花费 `minDist[u']` (新节点的 minDist) ≥ `minDist[u]` (我们选的最小). 加上后续边 (非负) → 路径长 ≥ `minDist[u]`. 所以现在的 `minDist[u]` 已是最短.

    > **关键前提: 边权非负**. 有负边权用 Bellman-Ford 或 SPFA.

5. **答案: `max(minDist[1..n])`, 任一为 INF 返 -1 / Max of all shortest distances**

    所有节点都要收到信号 → 取**最远那个**. 若任一节点 minDist 还是 INT_MAX → 不可达 → 返 -1.

6. **复杂度 O(n²) 稠密 / O((V+E) log V) 稀疏堆版 / Two implementations**

    - **Yang 的 O(n²)** (邻接矩阵 + 数组挑最小): 稠密图最优. 本题 n ≤ 100, ~10⁴ 操作.
    - **优先队列堆版** O((V + E) log V): 稀疏图更快. 用 `priority_queue<pair<int,int>, vector<...>, greater<>>` 出最小.

7. **跟 [0127 Word Ladder](../0127-word-ladder/README.md) 的对比 / vs BFS shortest path**

    | | 0127 BFS | **0743 Dijkstra** |
    |---|---|---|
    | 边权 | 全 1 (无权图) | 任意非负 |
    | 复杂度 | O(V + E) | O(n²) / O((V+E) log V) |
    | 取下一个 | FIFO 队列 | 最小 minDist |

    > **BFS 是 Dijkstra 在"全 1 边权" 下的特例**. 看清边权再决定用哪个.

## Solution

=== "C++"
    === "v1 推荐: O(n²) 稠密图 Dijkstra (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int networkDelayTime(vector<vector<int>>& times, int n, int k) {
                vector<int> minDist(n + 1, INT_MAX);
                vector<bool> visited(n + 1, false);
                vector<vector<int>> graph(n + 1, vector<int>(n + 1, INT_MAX));
                for (auto& t : times) graph[t[0]][t[1]] = t[2];
                minDist[k] = 0;

                for (int i = 1; i <= n; i++) {
                    // 挑当前 minDist 最小的未访问点
                    int u = -1, minVal = INT_MAX;
                    for (int v = 1; v <= n; v++) {
                        if (!visited[v] && minDist[v] < minVal) {
                            minVal = minDist[v];
                            u = v;
                        }
                    }
                    if (u == -1) break;                            // 剩下的不可达, 提前结束
                    visited[u] = true;

                    // 松弛: minDist[v] = min(minDist[v], minDist[u] + weight(u, v))
                    for (int v = 1; v <= n; v++) {
                        if (!visited[v] && graph[u][v] != INT_MAX
                            && minDist[u] + graph[u][v] < minDist[v]) {
                            minDist[v] = minDist[u] + graph[u][v];
                        }
                    }
                }
                int ans = 0;
                for (int v = 1; v <= n; v++) {
                    if (minDist[v] == INT_MAX) return -1;
                    ans = max(ans, minDist[v]);
                }
                return ans;
            }
        };
        ```

    === "v2: 堆版 Dijkstra (稀疏更优)"
        ```cpp
        class Solution {
        public:
            int networkDelayTime(vector<vector<int>>& times, int n, int k) {
                // 邻接表: graph[u] = [(v, w), ...]
                vector<vector<pair<int, int>>> graph(n + 1);
                for (auto& t : times) graph[t[0]].push_back({t[1], t[2]});

                vector<int> dist(n + 1, INT_MAX);
                dist[k] = 0;
                // 小顶堆: (dist, node), 取 dist 最小
                priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;
                pq.push({0, k});

                while (!pq.empty()) {
                    auto [d, u] = pq.top(); pq.pop();
                    if (d > dist[u]) continue;                     // 过期记录, 跳过
                    for (auto& [v, w] : graph[u]) {
                        if (dist[u] + w < dist[v]) {
                            dist[v] = dist[u] + w;
                            pq.push({dist[v], v});
                        }
                    }
                }

                int ans = 0;
                for (int v = 1; v <= n; v++) {
                    if (dist[v] == INT_MAX) return -1;
                    ans = max(ans, dist[v]);
                }
                return ans;
            }
        };
        ```

=== "Python"
    ```python
    import heapq

    class Solution:
        def networkDelayTime(self, times: list[list[int]], n: int, k: int) -> int:
            # 邻接表
            graph: list[list[tuple[int, int]]] = [[] for _ in range(n + 1)]
            for u, v, w in times:
                graph[u].append((v, w))

            INF = float('inf')
            dist = [INF] * (n + 1)
            dist[k] = 0
            # heapq 是小顶堆, (d, u) 按 d 排
            pq = [(0, k)]
            while pq:
                d, u = heapq.heappop(pq)
                if d > dist[u]:
                    continue
                for v, w in graph[u]:
                    if dist[u] + w < dist[v]:
                        dist[v] = dist[u] + w
                        heapq.heappush(pq, (dist[v], v))

            ans = max(dist[1:])
            return ans if ans < INF else -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} times
     * @param {number} n
     * @param {number} k
     * @return {number}
     */
    var networkDelayTime = function(times, n, k) {
        // JS 没原生 PriorityQueue, 这里给 O(n²) 数组版
        const graph = Array.from({length: n + 1}, () => new Array(n + 1).fill(Infinity));
        for (const [u, v, w] of times) graph[u][v] = w;
        const dist = new Array(n + 1).fill(Infinity);
        const visited = new Array(n + 1).fill(false);
        dist[k] = 0;

        for (let i = 1; i <= n; i++) {
            let u = -1, minVal = Infinity;
            for (let v = 1; v <= n; v++) {
                if (!visited[v] && dist[v] < minVal) {
                    minVal = dist[v];
                    u = v;
                }
            }
            if (u === -1) break;
            visited[u] = true;
            for (let v = 1; v <= n; v++) {
                if (!visited[v] && graph[u][v] !== Infinity
                    && dist[u] + graph[u][v] < dist[v]) {
                    dist[v] = dist[u] + graph[u][v];
                }
            }
        }
        let ans = 0;
        for (let v = 1; v <= n; v++) {
            if (dist[v] === Infinity) return -1;
            ans = Math.max(ans, dist[v]);
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(n²) (v1) / O((V + E) log V) (v2 堆版).
- **Space**: O(n²) (v1 邻接矩阵) / O(V + E) (v2 邻接表).

## 相关题目

- [1584. Min Cost to Connect All Points](../1584-min-cost-to-connect-all-points/README.md) — Prim's MST, 同款"挑最小 + 松弛"
- [0127. Word Ladder](../0127-word-ladder/README.md) — BFS 最短路 (无权图)
- [KC-94. 城市间货物运输 I](../kc-94-city-cargo-transport-i/README.md) — Bellman-Ford 入门 (可处理负边权)
- 0787\. Cheapest Flights Within K Stops (待补) — 带步数限制的 Dijkstra / Bellman-Ford
- 1631\. Path With Minimum Effort (待补) — 网格 Dijkstra (边权 = 高度差)
- 1976\. Number of Ways to Arrive at Destination (待补) — Dijkstra + 计数路径
- 0207\. Course Schedule (待补) — 拓扑排序判环 (DAG)
- [0210. Course Schedule II](../0210-course-schedule-ii/README.md) — 拓扑排序输出顺序
