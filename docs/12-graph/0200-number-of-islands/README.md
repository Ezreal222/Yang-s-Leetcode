# 0200. Number of Islands / 岛屿数量

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Grid, Flood Fill · 图, 深度优先, 广度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/number-of-islands/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 `0/1` 二维网格. `1` 表示陆地, `0` 表示水. **岛屿** = 上下左右相连的 `1` 形成的连通块. 求岛屿数量.

**中文**: 0/1 网格, 求 4-连通陆地块的个数.

## Key Insights

1. **🔑 网格 = 隐式图, 每个格子是节点, 4-邻居是边 / Grid as implicit graph**

    不用建邻接表 — 每个 `(i, j)` 的邻居就是 `(i±1, j)` 和 `(i, j±1)`. **岛屿 = 连通块**, 问题转化为"数连通块个数".

    > 看到"网格 + 连通" → 立刻反应"4-邻居 DFS / BFS / Union-Find". 这是网格题的通用范式.

2. **🔑 外层扫描 + 内层 flood fill / Outer scan + inner flood fill**

    模板:

    ```
    for each cell (i, j):
        if grid[i][j] == '1':
            count++
            dfs(i, j)   # 把整片岛屿 "染色"
    ```

    每次外层遇到 `'1'`, 计数 + 1, 然后 dfs 把这块岛**所有 `'1'` 标记为 `'0'`**, 让外层后续不会重复计数. 这就是"flood fill" (洪水填充).

3. **🔑 DFS 模板 — 边界 + 性质双重过滤 / DFS template: bounds + property check at top**

    Yang 的 DFS 第一行:

    ```cpp
    if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] != '1') return;
    ```

    一行四个条件: 越界 (4 个边界) + 不是 `'1'` (水或已访问). **写在函数入口** 比"进入前检查" 更省代码 — 每个调用点都不用判断, 进去后统一过滤.

    > **"top-down 边界过滤"** 是递归 DFS 的标配, 比"caller-side check" 干净.

4. **🔑 原地标记 `'1' → '0'` 当 visited / In-place visited via mutation**

    不用单独开 `visited` 数组. 把访问过的 `'1'` 直接改 `'0'`, 双重作用:

    - **标记已访问**: 下次外层扫到就跳过.
    - **避免 DFS 循环**: dfs 内邻居判 `grid[i][j] != '1'` 自动剔除已访问.

    缺点: **修改了输入**. 题目允许就这么干, 不允许就开 `visited` 数组.

5. **替代解法: BFS / Union-Find / Alternatives**

    | 解法 | 数据结构 | 优缺 |
    |---|---|---|
    | **DFS** (Yang) | 递归栈 | 最短最直观, 但深岛会栈溢出 |
    | **BFS** | 队列 | 无栈风险, 可早停 (本题不需要) |
    | **Union-Find** | 并查集 | 适合"动态加岛" (0305) 题型 |

    DFS 最常用. BFS 适合担心栈溢出的大网格 (本题 LC 数据 ≤ 300×300, DFS 够).

6. **复杂度 O(m × n) / Linear in cells**

    每个格子最多被访问一次 (访问后变 `'0'` 不再被访问). 总 O(m × n).

## Solution

=== "C++"
    === "v1 推荐: DFS + 原地标记 (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int numIslands(vector<vector<char>>& grid) {
                int count = 0;
                int m = grid.size(), n = grid[0].size();
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (grid[i][j] == '1') {
                            count++;
                            dfs(grid, i, j);                       // flood fill 整片岛
                        }
                    }
                }
                return count;
            }
        private:
            void dfs(vector<vector<char>>& grid, int i, int j) {
                int m = grid.size(), n = grid[0].size();
                // 一行四过滤: 越界 + 非陆地
                if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] != '1') return;
                grid[i][j] = '0';                                  // 原地标记 visited
                dfs(grid, i + 1, j);
                dfs(grid, i - 1, j);
                dfs(grid, i, j + 1);
                dfs(grid, i, j - 1);
            }
        };
        ```

    === "v2: BFS (无栈溢出风险)"
        ```cpp
        class Solution {
        public:
            int numIslands(vector<vector<char>>& grid) {
                int count = 0;
                int m = grid.size(), n = grid[0].size();
                vector<pair<int,int>> dirs = {{1,0},{-1,0},{0,1},{0,-1}};
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (grid[i][j] != '1') continue;
                        count++;
                        queue<pair<int,int>> q;
                        q.push({i, j});
                        grid[i][j] = '0';                          // 入队即标记
                        while (!q.empty()) {
                            auto [x, y] = q.front(); q.pop();
                            for (auto [dx, dy] : dirs) {
                                int nx = x + dx, ny = y + dy;
                                if (nx < 0 || nx >= m || ny < 0 || ny >= n
                                    || grid[nx][ny] != '1') continue;
                                grid[nx][ny] = '0';
                                q.push({nx, ny});
                            }
                        }
                    }
                }
                return count;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def numIslands(self, grid: list[list[str]]) -> int:
            if not grid:
                return 0
            m, n = len(grid), len(grid[0])
            count = 0

            def dfs(i: int, j: int) -> None:
                # 一行边界 + 性质过滤
                if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != '1':
                    return
                grid[i][j] = '0'                                   # 原地标记
                # 4 方向递归, Pythonic 用元组循环也行
                dfs(i + 1, j)
                dfs(i - 1, j)
                dfs(i, j + 1)
                dfs(i, j - 1)

            for i in range(m):
                for j in range(n):
                    if grid[i][j] == '1':
                        count += 1
                        dfs(i, j)
            return count
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} grid
     * @return {number}
     */
    var numIslands = function(grid) {
        const m = grid.length, n = grid[0].length;
        let count = 0;

        const dfs = (i, j) => {
            if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] !== '1') return;
            grid[i][j] = '0';
            dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1);
        };

        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] === '1') {
                    count++;
                    dfs(i, j);
                }
            }
        }
        return count;
    };
    ```

## Complexity

- **Time**: O(m × n) — 每格访问一次.
- **Space**: O(m × n) 最坏 (递归栈深度, 极端如"螺旋岛").

## 相关题目

- [0797. All Paths From Source to Target](../0797-all-paths-from-source-to-target/README.md) — DFS 显式图
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — 同模板, DFS 返回面积取 max
- [0463. Island Perimeter](../0463-island-perimeter/README.md) — 数岛屿周长, 不用 flood fill
- 1254\. Number of Closed Islands (待补) — 同模板, 过滤"挨边" 岛
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 反向 flood fill, 从边界出发标记安全区
- 0305\. Number of Islands II (待补) — 动态加岛 → Union-Find
- 0207\. Course Schedule (待补) — 有向图 + 拓扑排序
