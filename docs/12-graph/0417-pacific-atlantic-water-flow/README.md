# 0417. Pacific Atlantic Water Flow / 太平洋大西洋水流问题

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Grid, Flood Fill · 图, 深度优先, 广度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/pacific-atlantic-water-flow/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `m × n` 高度矩阵. 太平洋在**上 + 左** 两边, 大西洋在**下 + 右** 两边. 水从高处流向**不高于自己** 的相邻 (4-连通) 格子. 返回所有**水能同时流到两个海洋** 的格子坐标.

**中文**: 找出所有水既能流到太平洋 (上+左) 又能流到大西洋 (下+右) 的格子.

## Key Insights

1. **🔑 反向思考: "海洋能爬到哪些格子" 而不是"哪些格子能流到海洋" / Reverse the flow direction**

    正着想: 对每个格子, DFS 看能不能流到边界 — 每个格子要单独算, O(m² × n²), 太慢.

    **反向**: 从海洋边界出发**逆流而上** — 每次只走到**高度 ≥ 当前** 的邻居. 海洋能"爬到" 的格子, 就是"能流到海洋" 的格子.

    > 跟 [0130 包围区域](../0130-surrounded-regions/README.md) / [1020 飞地](../1020-number-of-enclaves/README.md) 同精神: **从答案的种子出发反向 flood fill, 比正向逐个检查省 n 倍**.

2. **🔑 两次独立 flood fill + 取交集 / Two flood fills, intersect at the end**

    - 维护两个 visited 矩阵: `pacific`, `atlantic`.
    - **从太平洋边界 (上 + 左) 出发**, DFS 标记 `pacific[i][j] = true`.
    - **从大西洋边界 (下 + 右) 出发**, DFS 标记 `atlantic[i][j] = true`.
    - 答案 = `pacific && atlantic` 的格子.

    > "两个集合的交集" 是这题的关键. 不能共用一个 visited — 必须分开记.

3. **🔑 "逆流" 转移条件: `h[neighbor] >= h[current]` / Uphill condition**

    水从高流低 = 反过来从低爬高 (或相等). DFS 从当前到邻居只走 `h[ni][nj] >= h[i][j]` 的方向. Yang 用 `if (h[ni][nj] < h[i][j]) continue` 排除"严格更低", 等价.

    > **等高也算能流** (题意默认), 所以是 `≥` 不是严格 `>`. 用错就漏算大量等高区.

4. **`visited` 标记防重复 / Visited prevents re-entry**

    标准 flood fill 标记. Yang 入口先判 `if (visited[i][j]) return`, 等价于 [0200](../0200-number-of-islands/README.md) 边界过滤的扩展. 两层防护 (入口 + 邻居前 check) 都写, 是稳健的写法.

5. **复杂度 O(m × n) / Linear**

    两遍 DFS, 每格各最多访问一次 (两个矩阵独立) → 2 × O(m × n) = O(m × n). 最终扫一遍取交集 O(m × n). 总线性.

6. **跟 [1020](../1020-number-of-enclaves/README.md) / [0130](../0130-surrounded-regions/README.md) 的差别 / vs 1020 / 0130**

    | | 1020 / 0130 | **0417 (本题)** |
    |---|---|---|
    | 入口集合 | **一个** (所有边界) | **两个** (太平洋边 + 大西洋边) |
    | 答案 | 单 flood fill 的"未到达" 或"已到达" | 两次 flood fill 的**交集** |
    | 边权约束 | 无 (任何连通就走) | 有 (高度 ≥ 才能爬) |

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int m, n;
        vector<vector<int>> h;
        vector<pair<int, int>> dirs = {{1,0}, {0,1}, {-1,0}, {0,-1}};

        vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
            h = heights;
            m = h.size(); n = h[0].size();
            vector<vector<bool>> pacific(m, vector<bool>(n, false));
            vector<vector<bool>> atlantic(m, vector<bool>(n, false));

            // 太平洋: 上 + 左
            for (int i = 0; i < m; i++) dfs(i, 0, pacific);
            for (int j = 0; j < n; j++) dfs(0, j, pacific);
            // 大西洋: 下 + 右
            for (int i = 0; i < m; i++) dfs(i, n - 1, atlantic);
            for (int j = 0; j < n; j++) dfs(m - 1, j, atlantic);

            // 取交集
            vector<vector<int>> ans;
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (pacific[i][j] && atlantic[i][j]) ans.push_back({i, j});
                }
            }
            return ans;
        }

    private:
        void dfs(int i, int j, vector<vector<bool>>& visited) {
            if (visited[i][j]) return;
            visited[i][j] = true;
            for (auto& [dx, dy] : dirs) {
                int ni = i + dx, nj = j + dy;
                if (ni < 0 || ni >= m || nj < 0 || nj >= n) continue;
                if (visited[ni][nj]) continue;
                if (h[ni][nj] < h[i][j]) continue;                  // ⚠ 反向逆流: 邻居必须 ≥ 当前
                dfs(ni, nj, visited);
            }
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def pacificAtlantic(self, heights: list[list[int]]) -> list[list[int]]:
            m, n = len(heights), len(heights[0])
            pacific = [[False] * n for _ in range(m)]
            atlantic = [[False] * n for _ in range(m)]
            dirs = [(1, 0), (0, 1), (-1, 0), (0, -1)]

            def dfs(i: int, j: int, visited: list[list[bool]]) -> None:
                if visited[i][j]:
                    return
                visited[i][j] = True
                for dx, dy in dirs:
                    ni, nj = i + dx, j + dy
                    if 0 <= ni < m and 0 <= nj < n and not visited[ni][nj] \
                            and heights[ni][nj] >= heights[i][j]:     # 逆流: ≥
                        dfs(ni, nj, visited)

            # 太平洋: 上 + 左
            for i in range(m): dfs(i, 0, pacific)
            for j in range(n): dfs(0, j, pacific)
            # 大西洋: 下 + 右
            for i in range(m): dfs(i, n - 1, atlantic)
            for j in range(n): dfs(m - 1, j, atlantic)

            # 取交集 — 列表推导一行
            return [[i, j] for i in range(m) for j in range(n)
                    if pacific[i][j] and atlantic[i][j]]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} heights
     * @return {number[][]}
     */
    var pacificAtlantic = function(heights) {
        const m = heights.length, n = heights[0].length;
        const pacific  = Array.from({length: m}, () => new Array(n).fill(false));
        const atlantic = Array.from({length: m}, () => new Array(n).fill(false));
        const dirs = [[1,0],[0,1],[-1,0],[0,-1]];

        const dfs = (i, j, visited) => {
            if (visited[i][j]) return;
            visited[i][j] = true;
            for (const [dx, dy] of dirs) {
                const ni = i + dx, nj = j + dy;
                if (ni < 0 || ni >= m || nj < 0 || nj >= n) continue;
                if (visited[ni][nj]) continue;
                if (heights[ni][nj] < heights[i][j]) continue;
                dfs(ni, nj, visited);
            }
        };

        for (let i = 0; i < m; i++) dfs(i, 0, pacific);
        for (let j = 0; j < n; j++) dfs(0, j, pacific);
        for (let i = 0; i < m; i++) dfs(i, n - 1, atlantic);
        for (let j = 0; j < n; j++) dfs(m - 1, j, atlantic);

        const ans = [];
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (pacific[i][j] && atlantic[i][j]) ans.push([i, j]);
            }
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(m × n) — 两遍 DFS 各 O(m × n), 最终扫一遍 O(m × n).
- **Space**: O(m × n) — 两个 visited + 递归栈.

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — flood fill 母题
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — DFS 返回面积
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — 单次反向 flood fill
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 单次反向 flood fill, 翻 X
- 1254\. Number of Closed Islands (待补) — 封闭岛计数
- 0286\. Walls and Gates (待补) — 多源 BFS, 从所有门出发
- [0463. Island Perimeter](../0463-island-perimeter/README.md) — 岛屿周长, 4-邻居贡献法
