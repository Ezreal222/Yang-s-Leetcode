# 0130. Surrounded Regions / 被围绕的区域

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Grid, Flood Fill · 图, 深度优先, 广度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/surrounded-regions/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `m × n` 网格 (`'X'` / `'O'`). 把所有**被 `'X'` 完全包围** (上下左右四周都被堵, 走不到边界) 的 `'O'` 全部翻成 `'X'`. **原地修改**.

**中文**: 把不挨边界的 `'O'` 区域全翻成 `'X'`, 原地改.

## Key Insights

1. **🔑 反向 flood fill: 标"安全 O", 剩下的翻 / Reverse flood fill: mark safe ones, flip the rest**

    "**被围绕**" = 不能通过 4-连通走到边界. 反过来想: **能走到边界的 `'O'` 就是"安全 O"**, 不会被翻; 不能走到边界的就要翻.

    > 跟 [1020 飞地](../1020-number-of-enclaves/README.md) 同精神. **正着想"哪些要变" 难, 反着想"哪些不变" 简单**.

2. **🔑 三步走 / Three-step recipe**

    1. **从所有边界 `'O'` 出发 DFS**, 把连通的 `'O'` 染成**临时标记 `'#'`** — 这些是"安全 O".
    2. **扫整张网格**:
        - `'#'` → `'O'` (恢复, 它本来就是 O, 没被围)
        - `'O'` → `'X'` (被围, 翻)
        - `'X'` 不动
    3. 完成原地修改.

    > 用**临时标记 `#`** 区分"已确认安全 O" 和"待判定 O" 是关键. 一遍 DFS + 一遍清扫就够.

3. **DFS 同 [0200](../0200-number-of-islands/README.md) / [1020](../1020-number-of-enclaves/README.md) 骨架 / Same DFS skeleton**

    - 边界 + 性质过滤: `board[i][j] != 'O'` 提前返回 (覆盖越界, 也覆盖 `'X'` 和 `'#'`).
    - 原地标记: `board[i][j] = '#'`.
    - 4 方向递归.

4. **🔑 为什么只 DFS 边界四条边? / Why DFS only from boundary**

    一个 `'O'` 安全当且仅当它和**某个边界 `'O'`** 4-连通. 所以从所有边界 `'O'` 出发洪水填充, 就能标完所有"安全 O". 中间的 `'O'` 不用主动 DFS — 走得到的, 已经被边界 DFS 找到.

    > **入口选对就省一半工**: 入口 = "确定属于答案的种子集合" → 反向 BFS/DFS.

5. **三符号设计的优雅 / Three-symbol encoding**

    | 符号 | 含义 |
    |---|---|
    | `'X'` | 本来就是 X, 永远不动 |
    | `'O'` | 待判定 — 第二遍变 X |
    | `'#'` | 确认安全 — 第二遍恢复 O |

    > 用**第三个临时符号** 区分"待判定" vs "已确认", 避免一次扫描里逻辑混乱. 是网格题里常见的小技巧.

6. **复杂度 O(m × n) / Linear**

    每格被 DFS 访问 O(1) 次 (因为染 `#` 后不再访问), 加最终扫一遍. 总 O(m × n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        void solve(vector<vector<char>>& board) {
            int m = board.size(), n = board[0].size();
            // 第一步: 从边界的 'O' 出发, 把所有"安全 O" 染成 '#'
            for (int i = 0; i < m; i++) {
                dfs(board, i, 0);
                dfs(board, i, n - 1);
            }
            for (int j = 0; j < n; j++) {
                dfs(board, 0, j);
                dfs(board, m - 1, j);
            }
            // 第二步: '#' → 'O' (恢复), 'O' → 'X' (翻)
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (board[i][j] == '#') board[i][j] = 'O';
                    else if (board[i][j] == 'O') board[i][j] = 'X';
                }
            }
        }
    private:
        void dfs(vector<vector<char>>& board, int i, int j) {
            int m = board.size(), n = board[0].size();
            // 边界 + 性质 (非 'O' 都不递归)
            if (i < 0 || i >= m || j < 0 || j >= n || board[i][j] != 'O') return;
            board[i][j] = '#';                                     // 标记安全
            dfs(board, i + 1, j); dfs(board, i - 1, j);
            dfs(board, i, j + 1); dfs(board, i, j - 1);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def solve(self, board: list[list[str]]) -> None:
            m, n = len(board), len(board[0])

            def dfs(i: int, j: int) -> None:
                if i < 0 or i >= m or j < 0 or j >= n or board[i][j] != 'O':
                    return
                board[i][j] = '#'                                  # 标记安全
                dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1)

            # 第一步: 边界 O 出发 DFS
            for i in range(m):
                dfs(i, 0); dfs(i, n - 1)
            for j in range(n):
                dfs(0, j); dfs(m - 1, j)

            # 第二步: '#' → 'O', 'O' → 'X'
            for i in range(m):
                for j in range(n):
                    if board[i][j] == '#':
                        board[i][j] = 'O'
                    elif board[i][j] == 'O':
                        board[i][j] = 'X'
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} board
     * @return {void} Do not return anything, modify board in-place instead.
     */
    var solve = function(board) {
        const m = board.length, n = board[0].length;

        const dfs = (i, j) => {
            if (i < 0 || i >= m || j < 0 || j >= n || board[i][j] !== 'O') return;
            board[i][j] = '#';
            dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1);
        };

        for (let i = 0; i < m; i++) {
            dfs(i, 0); dfs(i, n - 1);
        }
        for (let j = 0; j < n; j++) {
            dfs(0, j); dfs(m - 1, j);
        }

        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (board[i][j] === '#') board[i][j] = 'O';
                else if (board[i][j] === 'O') board[i][j] = 'X';
            }
        }
    };
    ```

## Complexity

- **Time**: O(m × n) — 每格 O(1) 次访问 (DFS 染 # 后不再访问).
- **Space**: O(m × n) 最坏 (递归栈).

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — flood fill 母题
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — flood fill 返回面积
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — **同款反向 flood fill**, 数飞地格子数
- [0417. Pacific Atlantic Water Flow](../0417-pacific-atlantic-water-flow/README.md) — 反向 flood fill 进阶, 两次从边出发取交集
- 1254\. Number of Closed Islands (待补) — 数封闭岛 (不挨边)
- 0286\. Walls and Gates (待补) — 多源 BFS, 从所有门出发
