# 0695. Max Area of Island / 岛屿的最大面积

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Grid, Flood Fill · 图, 深度优先, 广度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/max-area-of-island/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `0/1` 网格. 4-连通的 `1` 构成岛屿. 返回**最大岛屿的面积** (格子数). 没有岛屿返 `0`.

**中文**: 0/1 网格, 求最大 4-连通陆地块的格子数, 没有返 0.

## Key Insights

1. **🔑 [0200](../0200-number-of-islands/README.md) 的直接扩展 — DFS 改成返回面积 / Direct extension of 0200**

    [0200](../0200-number-of-islands/README.md) DFS 是 `void` (只染色), 本题 DFS 改成**返回 int** (染色 + 数格子数). 模板:

    ```
    dfs(i, j):
        if out of bounds or water: return 0
        mark visited
        return 1 + dfs(上下左右)
    ```

    外层用 `maxArea = max(maxArea, dfs(i, j))` 跟踪全局最大.

    > **"数 vs 数大小" 的小升级**: DFS 从 `void` 升 `int`, 累加 1 + 子树/邻居返回值. 跟二叉树高度 / 节点数模板一脉相承.

2. **🔑 DFS 返回值的累加: `1 + 四方向之和` / Recurrence: 1 + sum of 4 neighbors**

    每个 `1` 格子贡献 `1`, 然后递归四个邻居加起来. 越界 / 水返回 `0`, 自然剪枝.

    > 这跟"二叉树节点数" `count(root) = 1 + count(L) + count(R)` 同结构, 邻居数量从 2 升到 4.

3. **原地标记 visited (跟 0200 一样) / In-place visited**

    `grid[i][j] = 0` 标记访问过. 防止下一轮邻居再访问当前格子 (否则无限递归).

    > 这是网格 DFS 标准技巧, 见 [0200 Key Insight #4](../0200-number-of-islands/README.md).

4. **`maxArea = 0` 默认 / Default 0**

    没有任何 `1` 时 `dfs` 不会被调用, `maxArea` 保持 0. 跟"没有岛屿" 语义一致.

5. **复杂度 O(m × n) / Linear in cells**

    每格最多被访问一次. 跟 0200 同阶.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxAreaOfIsland(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            int maxArea = 0;
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (grid[i][j] == 1) {                         // 没必要检查访问过, dfs 会处理
                        maxArea = max(maxArea, dfs(grid, i, j));
                    }
                }
            }
            return maxArea;
        }
    private:
        int dfs(vector<vector<int>>& grid, int i, int j) {
            int m = grid.size(), n = grid[0].size();
            if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == 0) return 0;
            grid[i][j] = 0;                                        // 原地标记
            return 1
                 + dfs(grid, i + 1, j) + dfs(grid, i - 1, j)
                 + dfs(grid, i, j + 1) + dfs(grid, i, j - 1);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxAreaOfIsland(self, grid: list[list[int]]) -> int:
            m, n = len(grid), len(grid[0])

            def dfs(i: int, j: int) -> int:
                # 一行边界 + 性质过滤, 越界/水返 0 自然剪枝
                if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] == 0:
                    return 0
                grid[i][j] = 0                                     # 原地标记
                return 1 + dfs(i + 1, j) + dfs(i - 1, j) + dfs(i, j + 1) + dfs(i, j - 1)

            max_area = 0
            for i in range(m):
                for j in range(n):
                    if grid[i][j] == 1:
                        a = dfs(i, j)
                        if a > max_area:
                            max_area = a
            return max_area
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} grid
     * @return {number}
     */
    var maxAreaOfIsland = function(grid) {
        const m = grid.length, n = grid[0].length;
        let maxArea = 0;

        const dfs = (i, j) => {
            if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] === 0) return 0;
            grid[i][j] = 0;
            return 1 + dfs(i + 1, j) + dfs(i - 1, j) + dfs(i, j + 1) + dfs(i, j - 1);
        };

        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] === 1) {
                    maxArea = Math.max(maxArea, dfs(i, j));
                }
            }
        }
        return maxArea;
    };
    ```

## Complexity

- **Time**: O(m × n) — 每格访问一次.
- **Space**: O(m × n) 最坏 (递归栈深度).

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — 母题, 数岛屿个数 (DFS 不返回值)
- [0797. All Paths From Source to Target](../0797-all-paths-from-source-to-target/README.md) — DFS 显式图
- 0463\. Island Perimeter (待补) — 数岛屿周长, 边界 + 水的贡献
- 1254\. Number of Closed Islands (待补) — 过滤"挨边" 岛
- 0827\. Making A Large Island (待补) — 0695 进阶, 允许翻一块水
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 反向 flood fill, 从边界出发标记
