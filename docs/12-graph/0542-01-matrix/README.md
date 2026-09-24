# 0542. 01 Matrix / 01 矩阵

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: BFS, Multi-source BFS, Graph, DP · 广度优先, 多源 BFS, 图论, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/01-matrix/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Distance from every 1 to nearest 0** → **multi-source BFS starting from ALL zeros simultaneously**. Each 1 gets `parent + 1` when first visited. Use **-1 as unvisited** marker (dual-purpose: input flag + visited flag).
>
> **中文**: **每个 1 到最近 0 的距离** → **把所有 0 一起入队做多源 BFS**. 每个 1 首次被访问时 = 距离 = `父 + 1`. **用 -1 兼作"未访问" 标记** (省一个 visited 数组).
>
> *Template / 模版*: **Multi-source BFS from all targets, propagate outward** — 反向 BFS 招: 不从每个 1 独立 BFS (O(n²)), 而是**所有 0 同时入队**当"距离 0 层" (O(n)).

## Problem

**EN**: 01 矩阵 `mat`, 求每个 1 到**最近 0**的曼哈顿距离 (格子相邻). 0 距离自己 0.

**中文**: 求每个 1 到最近 0 的距离.

## Key Insights

1. **🔑 灵魂: "反向 BFS" — 所有 0 同时出发, 而非每个 1 单独跑 / Multi-source BFS from all zeros**

    **直觉错解**: 每个 1 单独 BFS 找最近 0 → **O((mn)²)** — 100×100 就 10⁸ 卡时间.

    **正解**: **把所有 0 一起入队** 当 BFS 的**起始层** (dist = 0). BFS 向外扩散, 首次触及每个 1 就是它的答案. **每格只访问 1 次** → **O(mn)**.

    → 想象每个 0 都同时喷水, 水波同步向外扩. 每个 1 被最先到达的水波染色, 那个染色值 = 距离.

    > **"求每格到 target 集的最短距离" 一律用多源 BFS**, 起始层 = target 集. 也见 [0994 Rotting Oranges](../0994-rotting-oranges/README.md), 0286 Walls and Gates (待补).

2. **🔑 灵魂: `-1` 一石二鸟 = 输入标记 + 未访问标记 / -1 dual-purpose**

    Yang 的巧:

    ```cpp
    if (mat[i][j] == 0) q.push({i, j});
    else mat[i][j] = -1;                    // 所有 1 改成 -1
    ```

    → **值域上**:
    - `0`: 距离 0 (自己是 0).
    - `-1`: 未访问 (原本是 1, 等待被赋距离).
    - `>0`: 已访问, 距离 = 该值.

    - 后面 BFS 判 `mat[nr][nc] != -1` 即"已访问, 跳过".
    - 首次到达时 `mat[nr][nc] = mat[r][c] + 1` **同时**赋距离 + 标记已访问.

    → **省一个 visited 数组** (O(mn) 空间). 用输入的语义空隙塞进 flag.

    > **"数组值当 flag" 是空间省法**. 前提: 值域有空隙. 见 [0442](../../01-array/0442-find-all-duplicates-in-an-array/README.md) 负号招, [0130](../0130-surrounded-regions/README.md) '#' 招.

3. **🔑 BFS 单调性: 首次到达 = 最短 / First visit = shortest**

    BFS 按**层** (距离) 扩散. 每格首次进队时的层号 = 从起始集到它的最短距离.

    → **一旦赋值就不再更新** (`!= -1` 就跳过). 保证 O(mn), 不会重复松弛.

    对比 Dijkstra: 边权都 1 时 BFS = Dijkstra 简化版.

    > **"BFS 天然求边权 = 1 的最短路"** 是图论常识.

4. **🔑 4 方向数组: 惯用招 / 4-direction array idiom**

    ```cpp
    int dirs[4][2] = {{0,1}, {0,-1}, {-1,0}, {1,0}};
    for (auto& d : dirs) { int nr = r + d[0], nc = c + d[1]; ... }
    ```

    - 比写 4 个 if 短很多.
    - 顺序不影响 BFS 正确性 (BFS 只关心层, 不关心方向).
    - 常见变体: 8 方向对角线, 骑士 8 方向.

    > **"方向数组 + for" 是网格 BFS/DFS 标准写法**. 面试脱口而出.

5. **🔑 边界判 4 个 / 4 boundary checks**

    ```cpp
    if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
    ```

    - 左上右下都判. 顺序无所谓, 短路即返.
    - 用 `continue` 而非 `if (in-bounds) { ... }` — 更扁平, 减嵌套.

    > **易错点 top 1**: 忘判 `nc >= n` 或方向数组写错. 每次写完过一遍 4 条件.

6. **🔑 队列元素: `pair<int, int>` / Queue payload**

    C++ `queue<pair<int, int>>`, 通过 `structured bindings` `auto [r, c] = q.front()` 展开. 现代 C++ 招, 比 `.first / .second` 好读.

    - Python: 用 `tuple` `(r, c)`, `collections.deque`.
    - JS: 用 `[r, c]` 数组, 二维破坏赋值.

7. **🔑 替代解法: DP 双向扫 / DP two-pass**

    | 方法 | Time | Space | 特点 |
    |---|---|---|---|
    | **多源 BFS** (Yang) | **O(mn)** | O(mn) 队 | 直觉自然 |
    | **DP 两遍扫** | **O(mn)** | O(1) 无 queue | in-place, 巧 |
    | 每个 1 单跑 BFS | O((mn)²) | 慢 | 淘汰 |

    DP 版:

    ```
    Pass 1 (top-left → bottom-right): mat[i][j] = min(mat[i-1][j], mat[i][j-1]) + 1
    Pass 2 (bottom-right → top-left): mat[i][j] = min(mat[i][j], mat[i+1][j] + 1, mat[i][j+1] + 1)
    ```

    → **两次线扫**同样 O(mn), **无队列**. 面试若追问 "O(1) 辅助空间" 可提.

8. **🔑 复杂度 / Complexity**

    - **Time**: O(m·n) — 每格入队 & 出队各 1 次.
    - **Space**: O(m·n) — 队最坏含全部格子.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
            int m = mat.size(), n = mat[0].size();
            queue<pair<int, int>> q;

            // 初始化: 所有 0 入队作起始层; 1 改 -1 兼作"未访问"
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (mat[i][j] == 0) q.push({i, j});
                    else                mat[i][j] = -1;
                }
            }

            int dirs[4][2] = {{0, 1}, {0, -1}, {-1, 0}, {1, 0}};

            // 多源 BFS: 每格首次访问 = 最短距离
            while (!q.empty()) {
                auto [r, c] = q.front(); q.pop();
                for (auto& d : dirs) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
                    if (mat[nr][nc] != -1) continue;                    // 已访问
                    mat[nr][nc] = mat[r][c] + 1;                        // 赋距离 = 父 + 1
                    q.push({nr, nc});
                }
            }
            return mat;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def updateMatrix(self, mat: list[list[int]]) -> list[list[int]]:
            m, n = len(mat), len(mat[0])
            # deque 是双端队列, appendleft/popleft O(1)
            # BFS 用 deque, 别用 list (list.pop(0) 是 O(n))
            q = deque()

            # 初始化: 所有 0 入队; 1 改 -1 作 unvisited flag
            for i in range(m):
                for j in range(n):
                    if mat[i][j] == 0:
                        q.append((i, j))
                    else:
                        mat[i][j] = -1

            # 4 方向: 惯用 tuple 列表, 比 [[0,1],[0,-1],...] 稍精简
            dirs = [(0, 1), (0, -1), (-1, 0), (1, 0)]

            while q:
                r, c = q.popleft()                # 左出右入 = FIFO
                for dr, dc in dirs:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < m and 0 <= nc < n and mat[nr][nc] == -1:
                        mat[nr][nc] = mat[r][c] + 1
                        q.append((nr, nc))
            return mat
    ```

=== "JavaScript"
    ```javascript
    var updateMatrix = function(mat) {
        const m = mat.length, n = mat[0].length;
        // JS 无原生队列, 用数组 push/shift; shift 是 O(n) 但 LC 数据规模能过
        // 追求性能可实现循环 buffer 或用 index 游标
        const q = [];

        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (mat[i][j] === 0) q.push([i, j]);
                else                 mat[i][j] = -1;
            }
        }

        const dirs = [[0, 1], [0, -1], [-1, 0], [1, 0]];

        while (q.length) {
            // 解构赋值: [r, c] = q.shift() 拿队首坐标
            const [r, c] = q.shift();
            for (const [dr, dc] of dirs) {
                const nr = r + dr, nc = c + dc;
                if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
                if (mat[nr][nc] !== -1) continue;
                mat[nr][nc] = mat[r][c] + 1;
                q.push([nr, nc]);
            }
        }
        return mat;
    };
    ```

=== "C++ (v2: DP 两遍扫, O(1) 辅助空间)"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
            int m = mat.size(), n = mat[0].size();
            const int INF = m + n;                          // 上界: 最远曼哈顿 ≤ m+n
            for (int i = 0; i < m; i++)
                for (int j = 0; j < n; j++)
                    if (mat[i][j] != 0) mat[i][j] = INF;    // 1 → INF, 0 保持

            // Pass 1: 只看上、左邻居
            for (int i = 0; i < m; i++)
                for (int j = 0; j < n; j++) {
                    if (i > 0) mat[i][j] = min(mat[i][j], mat[i-1][j] + 1);
                    if (j > 0) mat[i][j] = min(mat[i][j], mat[i][j-1] + 1);
                }
            // Pass 2: 只看下、右邻居
            for (int i = m - 1; i >= 0; i--)
                for (int j = n - 1; j >= 0; j--) {
                    if (i < m - 1) mat[i][j] = min(mat[i][j], mat[i+1][j] + 1);
                    if (j < n - 1) mat[i][j] = min(mat[i][j], mat[i][j+1] + 1);
                }
            return mat;
        }
    };
    ```

## Complexity

- **Time**: O(m · n).
- **Space**: O(m · n) BFS 队; DP 版 O(1).

## 相关题目

- [0994. Rotting Oranges](../0994-rotting-oranges/README.md) — 多源 BFS 母题, 找最短传染时间
- [0200. Number of Islands](../0200-number-of-islands/README.md) — 网格 DFS/BFS 基础
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 反向标记招
- [0417. Pacific Atlantic Water Flow](../0417-pacific-atlantic-water-flow/README.md) — 多源 BFS 从边界起
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — 边界 BFS 排除
- 0286\. Walls and Gates (待补) — 直接同款, 多源 BFS 求距离
- 1162\. As Far from Land as Possible (待补) — 多源 BFS 找最远岛
- 1091\. Shortest Path in Binary Matrix (待补) — BFS 求 8 方向最短路
- 0317\. Shortest Distance from All Buildings (待补) — 每栋楼 BFS 汇总
- 0934\. Shortest Bridge (待补) — 两岛间最短桥, BFS
