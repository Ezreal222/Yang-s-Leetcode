# 1334. Find the City With the Smallest Number of Neighbors at a Threshold Distance / 阈值距离内邻居最少的城市

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Floyd-Warshall, Shortest Path · 图, Floyd-Warshall, 最短路
    - **Link**: [LeetCode](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `n` 个城市无向带权图. 求**阈值距离内邻居数最少**的城市. 多解返**编号最大** 的.

**中文**: 全源最短路 + 邻居计数 + 取最少 (并列取大).

## Key Insights

1. **🔑 全源最短路 + 计数 → Floyd-Warshall 完美场景 / All-pairs SP + counting**

    跟 [KC-97 Floyd-Warshall](../kc-97-floyd-park/README.md) 一字不差的模板 — 区别只在**后处理**: 对每个城市数邻居 + 取 min.

    > **任何"问每个节点的某个全局属性" 的题** → Floyd 一次性算所有对的最短路, 后处理 O(N²).

2. **🔑 `INT_MAX / 2` 防溢出 / Use INT_MAX/2 not INT_MAX**

    Yang 用 `INT_MAX / 2` 当 INF. 为啥不是 INT_MAX?

    Floyd 内层 `dist[i][k] + dist[k][j]` — 若两边都是 INT_MAX 会**溢出成负数**, 然后 `min` 错把"溢出值" 当真实最短路.

    用 `INT_MAX / 2` 让两个 INF 相加仍 < INT_MAX, 不溢出.

    > 跟 KC-94 BF 的"`if (dist[from] != INT_MAX)` 守卫" 同思想 — 都是防 INF 算术. **Floyd 这边用更大哨兵更省事 (代码不用判 INF)**.

3. **🔑 Floyd 三层循环 — k 必须最外 / k outermost (recap)**

    跟 [KC-97](../kc-97-floyd-park/README.md) 同理: `dp[k][i][j] = 用编号 ≤ k 的中转点的最短` 的空间压缩, k 必须在最外保证用的是"上一轮 k - 1" 的值.

4. **🔑 并列取大: `cnt <= minNum` 不是 `<` / Tiebreak: `<=` for largest index wins**

    题目要求"多解返编号最大". Yang 写 `if (cnt <= minNum)` (注意是 `<=`): 编号大的城市后被遍历到, 若 cnt 相等也会覆盖 `ans`. 用严格 `<` 就漏题意.

    > **并列规则决定 `<` 还是 `<=`**. "返最早" 用 `<`, "返最晚" 用 `<=`. **每次写 min 类比较前问一次"题目要哪个"**.

5. **无向图边对称 / Undirected = bidirectional fill**

    `dist[e[0]][e[1]] = dist[e[1]][e[0]] = w`. 漏写一边就半图.

6. **对角线 `dist[i][i] = 0` 隐式处理 / Diagonal stays 0 implicitly**

    `dist[i][i]` 初始是 `INT_MAX / 2`. Floyd 跑到 `k = i` 时, `dist[i][i] = min(INT_MAX/2, dist[i][i] + dist[i][i])` 仍是 INT_MAX/2.

    后处理 `for j: if (j == i) continue;` 跳过自己, 不会把"自己" 算进邻居. **Yang 这层守卫是关键** — 否则 INT_MAX/2 不一定 > threshold (理论上是, 但写明确点更稳).

    > 更稳妥写法: 初始时显式 `for i: dist[i][i] = 0;`. 不影响答案, 概念更干净.

7. **复杂度 O(V³ + V²) / Cubic-bound**

    Floyd O(V³) + 计数 O(V²). LC 数据 `n ≤ 100` → 10⁶ 操作, 完全 OK.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
            // INT_MAX / 2 当 INF: 防 dist[i][k] + dist[k][j] 溢出
            vector<vector<int>> dist(n, vector<int>(n, INT_MAX / 2));
            for (auto& e : edges) {
                dist[e[0]][e[1]] = e[2];
                dist[e[1]][e[0]] = e[2];                               // 无向对称
            }

            // Floyd: k 最外
            for (int k = 0; k < n; k++) {
                for (int i = 0; i < n; i++) {
                    for (int j = 0; j < n; j++) {
                        dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                    }
                }
            }

            // 后处理: 对每个 i 数阈值内邻居; <= 保证并列取大编号
            int minNum = INT_MAX, ans = 0;
            for (int i = 0; i < n; i++) {
                int cnt = 0;
                for (int j = 0; j < n; j++) {
                    if (j == i) continue;
                    if (dist[i][j] <= distanceThreshold) cnt++;
                }
                if (cnt <= minNum) {                                    // ⚠ <= 不是 <
                    minNum = cnt;
                    ans = i;
                }
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findTheCity(self, n: int, edges: list[list[int]], distanceThreshold: int) -> int:
            INF = float('inf')
            dist = [[INF] * n for _ in range(n)]
            for i in range(n):
                dist[i][i] = 0                                          # 显式对角 0
            for u, v, w in edges:
                dist[u][v] = w
                dist[v][u] = w

            # Floyd
            for k in range(n):
                for i in range(n):
                    for j in range(n):
                        if dist[i][k] + dist[k][j] < dist[i][j]:
                            dist[i][j] = dist[i][k] + dist[k][j]

            # 后处理 — sum 生成器一行数邻居, 等价 C++ 内层 for
            min_num, ans = INF, 0
            for i in range(n):
                cnt = sum(1 for j in range(n) if j != i and dist[i][j] <= distanceThreshold)
                if cnt <= min_num:                                      # 并列取大编号
                    min_num = cnt
                    ans = i
            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @param {number[][]} edges
     * @param {number} distanceThreshold
     * @return {number}
     */
    var findTheCity = function(n, edges, distanceThreshold) {
        const INF = Number.MAX_SAFE_INTEGER / 2;
        const dist = Array.from({length: n}, () => new Array(n).fill(INF));
        for (let i = 0; i < n; i++) dist[i][i] = 0;
        for (const [u, v, w] of edges) {
            dist[u][v] = w;
            dist[v][u] = w;
        }
        for (let k = 0; k < n; k++) {
            for (let i = 0; i < n; i++) {
                for (let j = 0; j < n; j++) {
                    if (dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }
        let minNum = Infinity, ans = 0;
        for (let i = 0; i < n; i++) {
            let cnt = 0;
            for (let j = 0; j < n; j++) {
                if (j !== i && dist[i][j] <= distanceThreshold) cnt++;
            }
            if (cnt <= minNum) {
                minNum = cnt;
                ans = i;
            }
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(V³ + V²) = O(V³).
- **Space**: O(V²).

## 相关题目

- [KC-97. 小明逛公园 (Floyd)](../kc-97-floyd-park/README.md) — Floyd-Warshall 母模板 (KamaCoder 版)
- [0743. Network Delay Time](../0743-network-delay-time/README.md) — Dijkstra 单源
- [KC-94. 城市间货物运输 I](../kc-94-city-cargo-transport-i/README.md) — BF 单源
- 0399\. Evaluate Division (待补) — 带权图, Floyd / DFS / Union-Find 都行
- 2642\. Design Graph With Shortest Path Calculator (待补) — 设计题, 适合 Floyd
- 0815\. Bus Routes (待补) — BFS 最短路 (无权图)
