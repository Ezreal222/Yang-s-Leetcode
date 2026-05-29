# 1020. Number of Enclaves / 飞地的数量

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Grid, Flood Fill · 图, 深度优先, 广度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/number-of-enclaves/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `0/1` 网格. **飞地** = 不能通过 4-连通走到**网格边界** 的 `1`. 返回飞地的总数 (格子数).

**中文**: 求不能通过 4-连通走到边界的 1 的总数.

## Key Insights

1. **🔑 两种思路 — 同 flood-fill 骨架不同包装 / Two approaches, same skeleton**

    | 解法 | 思路 | 包装 |
    |---|---|---|
    | **v1**: 一次扫 + 跟踪标志 | DFS 整片岛, 边走边记 `touchBoundary`. 不挨边就把面积加入答案 | 需要"引用传递" 的 bool 标志 |
    | **v2 推荐**: 两次扫, 先去边再数 | **第一遍**: 从所有边界 `1` 出发 DFS, 把连通的 `1` 全染成 `0`. **第二遍**: 数剩下的 `1` | 不用标志, 思路更直观 |

    > **v2 是更 idiomatic 的"反向 flood fill"**: 把"不属于答案的部分" 先抹掉, 剩下的就是答案. 跟 0130 包围区域 / 0417 太平洋大西洋水流 (都待补) 同精神.

2. **状态: v1 用引用传 `bool& touchBoundary` / Reference flag for v1**

    DFS 走到边界格 (`i == 0 || i == m-1 || j == 0 || j == n-1`) 就置 `touchBoundary = true`. 因为引用, 整片岛 DFS 完后能正确反映"是否挨过边".

    > **传引用是 v1 的关键**. 传值就没用 — 各递归层独立改自己的副本.

3. **🔑 ⚠ 你代码里有个小 bug: 5 个 DFS 调用而非 4 / Stray self-call**

    Yang 的 return 行:

    ```cpp
    return 1 + dfs(grid, i, j, ...) + dfs(grid, i+1, j, ...) + ...
    //          ↑ 这个对 (i, j) 自己的调用是多余的
    ```

    第一个 `dfs(grid, i, j, ...)` 是对**当前格子自己** 的递归. `grid[i][j]` 刚被设为 `0`, 这个递归立刻返回 `0`. 不影响正确性, 但浪费一次调用. 修正版应该只有 4 个方向.

    > 调试技巧: 网格 DFS 写"`i±1, j` 或 `i, j±1`" 时检查每个 dfs 调用都改了**恰好一个**坐标. Yang 那行 `dfs(i, j, ...)` 没改任何坐标, 应该警觉.

4. **flood-fill 骨架跟 [0200](../0200-number-of-islands/README.md) / [0695](../0695-max-area-of-island/README.md) 一样 / Same skeleton**

    边界 + 性质 top-down 过滤, 原地标记 visited (`= 0`), 4 方向递归. 是 [§12 Graph](../index.md) 网格题的统一模板.

5. **复杂度 O(m × n) / Linear**

    每格最多访问一次. v1, v2 同阶.

## Solution

=== "C++"
    === "v2 推荐: 反向 flood fill (先去边, 再数)"
        ```cpp
        class Solution {
        public:
            int numEnclaves(vector<vector<int>>& grid) {
                int m = grid.size(), n = grid[0].size();
                // 第一遍: 从边界的 1 出发, 把所有挨边的岛染成 0
                for (int i = 0; i < m; i++) {
                    if (grid[i][0] == 1) dfs(grid, i, 0);
                    if (grid[i][n - 1] == 1) dfs(grid, i, n - 1);
                }
                for (int j = 0; j < n; j++) {
                    if (grid[0][j] == 1) dfs(grid, 0, j);
                    if (grid[m - 1][j] == 1) dfs(grid, m - 1, j);
                }
                // 第二遍: 数剩下的 1
                int count = 0;
                for (int i = 0; i < m; i++)
                    for (int j = 0; j < n; j++)
                        if (grid[i][j] == 1) count++;
                return count;
            }
        private:
            void dfs(vector<vector<int>>& grid, int i, int j) {
                int m = grid.size(), n = grid[0].size();
                if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == 0) return;
                grid[i][j] = 0;
                dfs(grid, i + 1, j); dfs(grid, i - 1, j);
                dfs(grid, i, j + 1); dfs(grid, i, j - 1);
            }
        };
        ```

    === "v1 (Yang 原版思路, 4 方向修正)"
        ```cpp
        class Solution {
        public:
            int numEnclaves(vector<vector<int>>& grid) {
                int m = grid.size(), n = grid[0].size();
                int result = 0;
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (grid[i][j] == 1) {
                            bool touchBoundary = false;
                            int area = dfs(grid, i, j, touchBoundary);
                            if (!touchBoundary) result += area;     // 不挨边才计入
                        }
                    }
                }
                return result;
            }
        private:
            int dfs(vector<vector<int>>& grid, int i, int j, bool& touchBoundary) {
                int m = grid.size(), n = grid[0].size();
                if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == 0) return 0;
                grid[i][j] = 0;
                // 走到边界格 → 标记挨边
                if (i == 0 || j == 0 || i == m - 1 || j == n - 1) touchBoundary = true;
                // ⚠ 修正: 4 个方向, 不要 5 个 (去掉了 dfs(i, j, ...) 自调用)
                return 1
                     + dfs(grid, i + 1, j, touchBoundary) + dfs(grid, i - 1, j, touchBoundary)
                     + dfs(grid, i, j + 1, touchBoundary) + dfs(grid, i, j - 1, touchBoundary);
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def numEnclaves(self, grid: list[list[int]]) -> int:
            # v2: 反向 flood fill
            m, n = len(grid), len(grid[0])

            def dfs(i: int, j: int) -> None:
                if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] == 0:
                    return
                grid[i][j] = 0
                dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1)

            # 第一遍: 从边界出发淹没挨边的岛
            for i in range(m):
                if grid[i][0] == 1: dfs(i, 0)
                if grid[i][n - 1] == 1: dfs(i, n - 1)
            for j in range(n):
                if grid[0][j] == 1: dfs(0, j)
                if grid[m - 1][j] == 1: dfs(m - 1, j)

            # 第二遍: 数剩下的
            return sum(grid[i][j] for i in range(m) for j in range(n))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} grid
     * @return {number}
     */
    var numEnclaves = function(grid) {
        const m = grid.length, n = grid[0].length;

        const dfs = (i, j) => {
            if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] === 0) return;
            grid[i][j] = 0;
            dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1);
        };

        for (let i = 0; i < m; i++) {
            if (grid[i][0] === 1) dfs(i, 0);
            if (grid[i][n - 1] === 1) dfs(i, n - 1);
        }
        for (let j = 0; j < n; j++) {
            if (grid[0][j] === 1) dfs(0, j);
            if (grid[m - 1][j] === 1) dfs(m - 1, j);
        }

        let count = 0;
        for (let i = 0; i < m; i++)
            for (let j = 0; j < n; j++)
                if (grid[i][j] === 1) count++;
        return count;
    };
    ```

## Complexity

- **Time**: O(m × n) — 每格访问 O(1) 次 (v2 两遍, v1 一遍 + flag 检查).
- **Space**: O(m × n) 最坏 (递归栈).

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — 数岛屿个数
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — 最大岛屿面积
- 0130\. Surrounded Regions (待补) — 同款"反向 flood fill", 从边界出发标记
- 0417\. Pacific Atlantic Water Flow (待补) — 反向 flood fill 进阶, 两次出发
- 1254\. Number of Closed Islands (待补) — 数封闭岛 (不挨边)
- 1905\. Count Sub Islands (待补) — 子岛屿计数
