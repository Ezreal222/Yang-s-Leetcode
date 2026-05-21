# 0079. Word Search / 单词搜索

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, DFS, Grid · 回溯, 深度优先搜索, 网格
    - **Link**: [LeetCode](https://leetcode.com/problems/word-search/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given an `m × n` grid of characters and a `word`, return whether `word` exists in the grid via cells connected by 4-directional adjacency (each cell used at most once).

**中文**: 给一个 `m × n` 字符网格和一个 `word`, 判断能否在网格里通过 4 方向相邻 (每格最多用一次) 拼出 `word`.

## 思路

### Core idea

外层双 for 找起点; 内层 DFS 在 (i, j, idx) 上推进:

- `idx == word.size()` → 整词匹配, return true.
- 越界 / 字符不等 → return false.
- **In-place 标记访问**: `board[i][j] = '#'` 暂时占位, 4 方向 dfs 用 `||` 短路求 OR.
- 出栈前 **`board[i][j] = temp`** 还原 — 这就是显式回溯, 让兄弟分支 / 别的起点能用同一格.

### Key Insights

1. **In-place mark + temp restore = 省 visited[][] / Yang 的关键技巧**

    传统写法: 单独维护 `vector<vector<bool>> visited`, 进入时标 true, 退出时标 false. 多 O(mn) 空间, 多两步赋值.

    Yang 这版用**特殊字符 `'#'` 直接覆盖** `board[i][j]`, 退出前从 `temp` 恢复. **没有任何额外空间**, 只是临时改原始数据.

    适用条件: 特殊符号必须**不在原 board 字符集**里 (题目只用字母, `'#'` 安全). 这是 grid DFS / 回溯题常用的小优化, 值得记.

2. **`||` 短路 4 方向 = 不必显式 (dr, dc) for / Concise direction OR**

    ```cpp
    dfs(i+1,j) || dfs(i-1,j) || dfs(i,j+1) || dfs(i,j-1)
    ```
    任一方向找到 → 整体 true → 后面的 dfs 自动不执行 (短路求值). 比写 `for ((dr, dc) : {{1,0},{-1,0},{0,1},{0,-1}})` 4 元组循环少几行, 也快一点 (省 vector 构造).

3. **Terminal `idx == size` 必须在越界判前 / Order matters**

    顺序: **先判完成**, 再判越界, 再判字符不等. 想反就错: 若 word 最后一个字符匹配的格子正好在边界, 顺利匹配后下一层 dfs 会越界, 此时 `idx == size` 早已该 return true — 必须先检查.

4. **跟其它 grid DFS 模板的对照 / Grid DFS family**

    | 题 | 标记策略 | 是否回溯 |
    |---|---|---|
    | **0079 (本题)** | board[i][j] = '#' + temp | ✅ 退出还原 |
    | 0200 Number of Islands (待补) | board[i][j] = '0' 或 visited[][] | ❌ 永久标记 (一遍 sweep) |
    | 0463 Island Perimeter (待补) | 不标记, 数边 | — |
    | 0212 Word Search II (待补) | 0079 + Trie 同时搜多词 | ✅ |

    关键区分: **搜路径** (要回溯, 同一格可重新成为别路径一员) vs **数连通分量** (永久标记, 不必回溯).

5. **复杂度 O(mn · 4^L) / Exponential but small in practice**

    L = word.length, mn 起点, 每起点最坏 4^L 分支. 字符不等剪枝实际飞快.

6. **0051 / 0037 / 0079 都是 bool 短路 + 显式回溯 grid 类 / Hard-回溯 family extension**

    Yang 现在见过 4 个 grid + 回溯题: [0051](../0051-n-queens/README.md) (求所有, void), [0037](../0037-sudoku-solver/README.md) (bool 短路), [0473](../0473-matchsticks-to-square/README.md) (bool 短路 + k 桶), **0079 (本题, bool 短路 + 4 方向 + in-place 标记)**. 4 个套路同骨架, 状态/输出/标记策略各异.

### 一句话总结

**外层 for 找起点, dfs 在 (i, j, idx) 推进. board[i][j] = '#' in-place 标记访问, temp 还原回溯. 4 方向 `||` 短路 OR.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool dfs(vector<vector<char>>& board, const string& word, int i, int j, int idx) {
            if (idx == (int)word.size()) return true;                       // 必须最先判: 整词匹配完成
            if (i < 0 || i >= (int)board.size() ||
                j < 0 || j >= (int)board[0].size()) return false;           // 越界
            if (board[i][j] != word[idx]) return false;                     // 字符不等

            char temp = board[i][j];
            board[i][j] = '#';                                              // in-place 标记访问

            bool found = dfs(board, word, i + 1, j, idx + 1)
                      || dfs(board, word, i - 1, j, idx + 1)
                      || dfs(board, word, i, j + 1, idx + 1)
                      || dfs(board, word, i, j - 1, idx + 1);

            board[i][j] = temp;                                             // 还原: 让其他路径能用此格
            return found;
        }
        bool exist(vector<vector<char>>& board, string word) {
            for (int i = 0; i < (int)board.size(); i++)
                for (int j = 0; j < (int)board[0].size(); j++)
                    if (dfs(board, word, i, j, 0)) return true;
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def exist(self, board: list[list[str]], word: str) -> bool:
            m, n = len(board), len(board[0])
            def dfs(i: int, j: int, idx: int) -> bool:
                if idx == len(word):
                    return True
                if i < 0 or i >= m or j < 0 or j >= n:
                    return False
                if board[i][j] != word[idx]:
                    return False
                temp, board[i][j] = board[i][j], '#'                        # 同 C++ 临时存 + 占位
                # any() + 生成器表达式 = 短路 OR. 等价 C++ ||, 一旦找到 True 立刻停
                found = any(dfs(i + di, j + dj, idx + 1)
                            for di, dj in ((1, 0), (-1, 0), (0, 1), (0, -1)))
                board[i][j] = temp                                          # 还原
                return found
            return any(dfs(i, j, 0) for i in range(m) for j in range(n))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} board
     * @param {string} word
     * @return {boolean}
     */
    var exist = function(board, word) {
        const m = board.length, n = board[0].length;
        const dfs = (i, j, idx) => {
            if (idx === word.length) return true;
            if (i < 0 || i >= m || j < 0 || j >= n) return false;
            if (board[i][j] !== word[idx]) return false;
            const temp = board[i][j];
            board[i][j] = '#';
            const found = dfs(i + 1, j, idx + 1)
                       || dfs(i - 1, j, idx + 1)
                       || dfs(i, j + 1, idx + 1)
                       || dfs(i, j - 1, idx + 1);
            board[i][j] = temp;
            return found;
        };
        for (let i = 0; i < m; i++)
            for (let j = 0; j < n; j++)
                if (dfs(i, j, 0)) return true;
        return false;
    };
    ```

## Complexity

- **Time**: O(mn · 4^L), L = word.length. 实际剪枝后远小.
- **Space**: O(L) recursion depth. **不需要 visited 数组** (in-place 标记).

## 易错点

- **board 必须 restore (`board[i][j] = temp`)**: 漏一行整道题废 — 该格永久标 `'#'`, 其它路径再访问会以为字符不匹配, 大量答案丢失.
- **terminal `idx == size` 必须先判**: 顺序错会在 word 最后一个字符成功匹配后, 因下一层越界而返 false. 三层 if 顺序 = 完成 → 越界 → 字符.

## 相关题目

- [0051. N-Queens](../0051-n-queens/README.md) / [0037. Sudoku Solver](../0037-sudoku-solver/README.md) — 同款 grid + 显式回溯
- [0473. Matchsticks to Square](../0473-matchsticks-to-square/README.md) — 同款 bool 短路 + 显式回溯
- 0200\. Number of Islands (待补) — 同款 grid DFS, 但不需要回溯 (永久标记)
- 0212\. Word Search II (待补) — 多词搜索, 0079 + Trie 共享前缀剪枝
- 0463\. Island Perimeter (待补) — 同款 grid 遍历, 数边而非搜路径
- 0130\. Surrounded Regions (待补) — 同款 grid DFS 反向标记 (从边界反推)
