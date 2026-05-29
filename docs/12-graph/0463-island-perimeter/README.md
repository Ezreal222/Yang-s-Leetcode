# 0463. Island Perimeter / 岛屿的周长

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Graph, Grid, Math · 图, 网格, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/island-perimeter/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `0/1` 网格 (题目保证**恰好一个岛**). 每个 `1` 是边长 1 的正方形陆地, 求**岛的周长**.

**中文**: 求恰好一个岛屿的周长.

## Key Insights

1. **🔑 不用 DFS — 局部统计就够 / No DFS needed, local counting suffices**

    周长 = 所有"陆地格子" 的"暴露边" 总和. 每个 `1` 看四方向: 邻居是**水 或 越界** → 那条边算 1 单位周长.

    > **网格 + 数边长 / 数邻居** 类题往往不需要遍历连通块. **每个格子独立贡献** 即可.

2. **v1 直接法: 对每个陆地数暴露边 / Direct: count exposed sides per land cell**

    四邻居遍历, 计数. 简单直观. 复杂度 O(m × n × 4) = O(m × n).

3. **🔑 v2 数学法: `周长 = 4 × 陆地数 - 2 × 相邻陆地对数` / Formula approach**

    每个 `1` 单独贡献 4 条边. 但每对**相邻陆地** 之间共享 1 条边 — 这条边不在周长上, 但被两边各算了 1 次, 共需**减 2**.

    $$\text{Perimeter} = 4 \times \text{land} - 2 \times \text{neighbor\_pairs}$$

    **只查 (i+1, j) 和 (i, j+1) 两个方向** 数 pair, 不重复 (每对只被遍历到一次).

    > **"4 - 2k" 推导** 是这题的精髓: 把"边" 分两类 (边界边 / 内部共享边), 分别用乘法原理算. 跟 0084/0907 的"贡献法" 同精神, 但更轻量.

4. **不用 visited / 不用 DFS — 因为题目保证只有一个岛 / Single island assumption**

    题目明确说"只有一个岛", 我们不需要区分多个岛或避免重复. 直接全局扫一遍统计就行.

    > 若题目改成"多个岛求总周长", 公式法**仍然成立** (每个岛各自贡献), 直接答案. **公式法在"多岛" 场景反而比 DFS 更优**.

5. **复杂度 O(m × n) 两版都是 / Both O(m × n)**

    v1 每格 4 邻居查; v2 每格 2 邻居查. 常数差一倍, 量级一样.

6. **跟 [0200](../0200-number-of-islands/README.md) / [0695](../0695-max-area-of-island/README.md) 的对比 / vs Flood-fill family**

    | 题 | 算什么 | 解法 |
    |---|---|---|
    | [0200](../0200-number-of-islands/README.md) | 岛屿**个数** | DFS flood fill |
    | [0695](../0695-max-area-of-island/README.md) | 最大岛**面积** | DFS 返回面积 |
    | [0827](../0827-making-a-large-island/README.md) | 翻 1 块水后最大面积 | 标 ID + 查表 |
    | **0463 (本题)** | **周长** | **局部统计 (不需要 DFS)** |

    > **"算什么" 决定要不要遍历连通块**. 周长是局部属性 (每格独立贡献), 数岛 / 算面积才需要遍历.

## Solution

=== "C++"
    === "v2 推荐: 公式法 `4L - 2N`"
        ```cpp
        class Solution {
        public:
            int islandPerimeter(vector<vector<int>>& grid) {
                int m = grid.size(), n = grid[0].size();
                int land = 0, neighbors = 0;
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (grid[i][j] != 1) continue;
                        land++;
                        // 只查右 + 下, 避免重复数 pair
                        if (i + 1 < m && grid[i + 1][j] == 1) neighbors++;
                        if (j + 1 < n && grid[i][j + 1] == 1) neighbors++;
                    }
                }
                return 4 * land - 2 * neighbors;
            }
        };
        ```

    === "v1: 直接数暴露边"
        ```cpp
        class Solution {
        public:
            int islandPerimeter(vector<vector<int>>& grid) {
                int m = grid.size(), n = grid[0].size();
                vector<pair<int, int>> dirs = {{1,0}, {0,1}, {-1,0}, {0,-1}};
                int perimeter = 0;
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (grid[i][j] != 1) continue;
                        for (auto& [dx, dy] : dirs) {
                            int ni = i + dx, nj = j + dy;
                            // 越界 or 水 → 这条边露出来了
                            if (ni < 0 || ni >= m || nj < 0 || nj >= n || grid[ni][nj] == 0) {
                                perimeter++;
                            }
                        }
                    }
                }
                return perimeter;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def islandPerimeter(self, grid: list[list[int]]) -> int:
            m, n = len(grid), len(grid[0])
            # v2 公式法
            land = neighbors = 0
            for i in range(m):
                for j in range(n):
                    if grid[i][j] != 1:
                        continue
                    land += 1
                    if i + 1 < m and grid[i + 1][j] == 1:
                        neighbors += 1
                    if j + 1 < n and grid[i][j + 1] == 1:
                        neighbors += 1
            return 4 * land - 2 * neighbors
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} grid
     * @return {number}
     */
    var islandPerimeter = function(grid) {
        const m = grid.length, n = grid[0].length;
        let land = 0, neighbors = 0;
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] !== 1) continue;
                land++;
                if (i + 1 < m && grid[i + 1][j] === 1) neighbors++;
                if (j + 1 < n && grid[i][j + 1] === 1) neighbors++;
            }
        }
        return 4 * land - 2 * neighbors;
    };
    ```

## Complexity

- **Time**: O(m × n) — 两版都是.
- **Space**: O(1) — 不用额外结构.

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — flood fill 母题
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — 最大面积
- [0827. Making A Large Island](../0827-making-a-large-island/README.md) — 翻水版
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — 反向 flood fill
- 1254\. Number of Closed Islands (待补) — 封闭岛
- 1905\. Count Sub Islands (待补) — 子岛屿
