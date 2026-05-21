# 0052. N-Queens II / N 皇后 II

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Backtracking, Board, Recursion · 回溯, 棋盘, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/n-queens-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given an integer `n`, return the **number** of distinct solutions to the n-queens puzzle.

**中文**: 给整数 `n`, 返回 n 皇后问题的**解的数量**.

## 思路

### Core idea

**完全是 [0051](../0051-n-queens/README.md) 的"只数不收"版**. 同骨架 (逐行放 + isValid 上方三方向 + 显式 push/pop), 唯一改动:

```cpp
// 0051:
if (row == n) { res.push_back(board); return; }

// 0052:
if (row == n) { res++; return; }
```

输出语义从"收集所有 board" 改成"累加计数".

### Key Insights

1. **回溯输出语义的第三种形态 / Third output mode**

    Yang 现在见过的所有回溯输出语义:

    | 模式 | 例 | 返回 |
    |---|---|---|
    | 收集所有解 | [0051](../0051-n-queens/README.md) / [0078](../0078-subsets/README.md) | `vector<vector<...>> res` |
    | 找任一解 (短路) | [0037](../0037-sudoku-solver/README.md) | `bool` 一路 return |
    | **只数解数** (本题) | **0052** | `int res` 累加 |

    第三种是 0051 → 0052 的直接 simplification. 不需要保留 board (只是为了 isValid 的副作用), 但要保留递归结构.

2. **进阶: 完全扔掉 board / Drop the board entirely**

    既然只数不收, board 唯一作用是给 isValid 查冲突. 用 3 个 bool 数组替代 isValid (同 [0051 的进阶版](../0051-n-queens/README.md#solution)):

    ```cpp
    vector<bool> cols(n), d1(2*n), d2(2*n);
    function<void(int)> dfs = [&](int row) {
        if (row == n) { res++; return; }
        for (int col = 0; col < n; col++) {
            if (cols[col] || d1[row + col] || d2[row - col + n]) continue;
            cols[col] = d1[row + col] = d2[row - col + n] = true;
            dfs(row + 1);
            cols[col] = d1[row + col] = d2[row - col + n] = false;
        }
    };
    dfs(0);
    ```
    省掉 O(n²) board 内存 + 每次 isValid 从 O(n) 降到 O(1). N=14 之后差距非常大.

3. **极致优化: 位运算 / Bit-tricks for ~10× speed**

    一个 `int` (n ≤ 9 时一行能 fit) 表示行内"哪些列被屏蔽". `(cols | d1 | d2)` 直接拿到屏蔽掩码, `~mask & ((1 << n) - 1)` 拿可用列, **Brian Kernighan trick** `x & -x` 取最低位枚举. 经典 N=8 实现可以 1ms 内.

4. **为什么不能 DP / Why no memoization here**

    数独/N-皇后 的状态都是"目前棋盘 + 当前行", 状态空间巨大且**几乎不会重复** — 没有重叠子问题 → DP 不适用. 这是回溯 vs DP 的本质分界.

### 一句话总结

**0051 同款骨架, 收果实改成 `res++`. 想极致快用 3 bool 数组扔掉 board, 或位运算.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int res = 0;
        bool isValid(vector<string>& board, int row, int col, int n) {
            for (int i = 0; i < row; i++)
                if (board[i][col] == 'Q') return false;
            for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--)
                if (board[i][j] == 'Q') return false;
            for (int i = row - 1, j = col + 1; i >= 0 && j < n;  i--, j++)
                if (board[i][j] == 'Q') return false;
            return true;
        }
        void backtrack(vector<string>& board, int row, int n) {
            if (row == n) {
                res++;                                           // 唯一跟 0051 的差别
                return;
            }
            for (int col = 0; col < n; col++) {
                if (!isValid(board, row, col, n)) continue;
                board[row][col] = 'Q';
                backtrack(board, row + 1, n);
                board[row][col] = '.';
            }
        }
        int totalNQueens(int n) {
            vector<string> board(n, string(n, '.'));
            backtrack(board, 0, n);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def totalNQueens(self, n: int) -> int:
            # 进阶版: 不维护 board, 直接 3 个 set 存"已占用的 col / 主对角 / 副对角"
            # 比 C++ 那版本更短, 也是 LC 标准 Pythonic 写法
            cols, diag1, diag2 = set(), set(), set()
            count = 0
            def backtrack(row: int):
                nonlocal count
                if row == n:
                    count += 1
                    return
                for col in range(n):
                    # diag1 = row + col (↘ 主对角线 id), diag2 = row - col (↗ 副对角线 id)
                    if col in cols or (row + col) in diag1 or (row - col) in diag2:
                        continue
                    cols.add(col); diag1.add(row + col); diag2.add(row - col)
                    backtrack(row + 1)
                    cols.remove(col); diag1.remove(row + col); diag2.remove(row - col)
            backtrack(0)
            return count
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var totalNQueens = function(n) {
        // 同 Python 版本: 3 个 Set 存占用, 不维护 board
        const cols = new Set(), d1 = new Set(), d2 = new Set();
        let count = 0;
        const backtrack = (row) => {
            if (row === n) { count++; return; }
            for (let col = 0; col < n; col++) {
                if (cols.has(col) || d1.has(row + col) || d2.has(row - col)) continue;
                cols.add(col); d1.add(row + col); d2.add(row - col);
                backtrack(row + 1);
                cols.delete(col); d1.delete(row + col); d2.delete(row - col);   // Set.delete, 等价 Python remove
            }
        };
        backtrack(0);
        return count;
    };
    ```

## Complexity

- **Time**: O(n!·n) (基础版) / O(n!) (3 数组进阶版).
- **Space**: O(n²) board (基础) 或 O(n) (进阶) + O(n) recursion.

## 易错点

- **`res` 跨调用可能不重置**: LC 通常给 fresh 实例, 但题目接口若复用 Solution 对象, `res` 会累加. 安全做法: 入口 `res = 0;` 或者用 `count` 局部变量 (Python/JS 版那样).
- **不要 DP / 记忆化**: 状态唯一, 没重叠子问题. 看到"数解数"就以为是 DP 是常见误导.

## 相关题目

- [0051. N-Queens](../0051-n-queens/README.md) — 收集所有解的版本, 本题去掉 push_back 改成 `res++`
- [0037. Sudoku Solver](../0037-sudoku-solver/README.md) — 同款棋盘 + isValid, 但 bool 短路求任一解
- [0046. Permutations](../0046-permutations/README.md) — N-Queens 也可看作"列的排列, 加对角线约束"
- 0473\. Matchsticks to Square (待补) — 经典回溯 + 4-way 分桶, 同款"求是否能 + 回溯"
