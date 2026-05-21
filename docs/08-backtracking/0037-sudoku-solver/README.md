# 0037. Sudoku Solver / 解数独

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Backtracking, Board, Recursion · 回溯, 棋盘, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/sudoku-solver/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Fill a partially-filled 9×9 Sudoku board (`'.'` = empty) in place. Every row, column, and 3×3 box must contain digits 1-9 exactly once. There's guaranteed to be a unique solution.

**中文**: 在 9×9 数独棋盘上原地填数 (`'.'` 表示空格). 每行/每列/每 3×3 宫格必须含 1-9 各一次. 题目保证有唯一解.

## 思路

### Core idea

三层 for 循环 (Yang 的总结很到位):

- **第 1 个 for `i`**: 行.
- **第 2 个 for `j`**: 列. 跳过非空格 (`continue`).
- **第 3 个 for `num`**: 1-9 试. 合法就放, 递归; 递归 true 立即返 true; 失败就撤.

**bool 返回 + 短路**: 找到任一解就一路 `return true` 传到顶, 不需要继续探索. 跟 [0051](../0051-n-queens/README.md) 求所有解的 void 模板对比鲜明.

### Key Insights

1. **bool 返回值的"短路"语义 / Return-true-to-stop**

    | 题 | 找几个解 | 返回类型 | 行为 |
    |---|---|---|---|
    | [0051](../0051-n-queens/README.md) N-Queens | **所有** | void | 找到一个 push 进 res, 继续探索 |
    | **0037 (本题)** | **唯一一个** | **bool** | 找到立刻 `return true`, 顶层一路接住 |

    一句话: **求所有用 void + 全局收集; 求第一个用 bool + 短路**. 这是回溯系列的"输出语义"轴.

2. **找下一个空格 + for 9 数字 / Locate next empty, try 1-9**

    `if (board[i][j] != '.') continue;` 跳过已填. 找到一个空格就 for 1-9. **关键**: 9 个数字都不行时, `return false` **从函数出去**, 不继续遍历后面的格子 — 因为如果当前格子无解, 整张棋盘当前状态就无解, 任何后续的位置改动都救不回来.

    全部格子非空 (双重 for 跑完) → `return true`.

3. **isValid 三方向 / Three conflict checks**

    - **同行**: 9 个 `board[row][j]`.
    - **同列**: 9 个 `board[i][col]`.
    - **同 3×3 box**: `boxRow = (row/3)*3, boxCol = (col/3)*3`. 整除技巧把 0/1/2 → 0, 3/4/5 → 3, 6/7/8 → 6, 自动定位到 box 的左上角.

4. **跟 [0051 N-Queens 的对比 / Compared to N-Queens**

    | 项 | 0051 | 0037 |
    |---|---|---|
    | 推进 | 下一行 | 下一个空格 (本质等价 row × col 两层 for) |
    | isValid | 同列向上 + 两斜上 | 同行 + 同列 + 同 box |
    | 返回 | void, 累积 res | **bool**, 短路 |
    | 何时收 | row == n | 全部格非空时 |

    思路同源 (棋盘 + 显式 push/pop + 局部冲突检查), 输出语义不同.

5. **进阶: MRV + bitmask / Standard pro optimizations**

    - **MRV (Minimum Remaining Values)**: 每步挑"候选最少" 的空格优先填. 极大剪枝, 是数独求解器的工业级技巧.
    - **9 个 bitmask** 存"行/列/box 用过的数字" → isValid O(1) 查 + 一行更新. 总 O(N²) 预处理 + 极快.

    LC 数据不必, 但是面试常被问的标准 follow-up.

### 一句话总结

**三层 for (行 / 列 / 1-9 数字), isValid 查同行+同列+同 3×3 box. bool 返回短路, 找到一解就一路传 true.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isValid(vector<vector<char>>& board, int row, int col, char num) {
            for (int j = 0; j < 9; j++)                                      // 同行
                if (board[row][j] == num) return false;
            for (int i = 0; i < 9; i++)                                      // 同列
                if (board[i][col] == num) return false;
            int boxRow = (row / 3) * 3, boxCol = (col / 3) * 3;              // 整除定位 box 左上角
            for (int i = boxRow; i < boxRow + 3; i++)
                for (int j = boxCol; j < boxCol + 3; j++)
                    if (board[i][j] == num) return false;
            return true;
        }
        bool backtrack(vector<vector<char>>& board) {
            for (int i = 0; i < 9; i++) {
                for (int j = 0; j < 9; j++) {
                    if (board[i][j] != '.') continue;                        // 跳过已填
                    for (char num = '1'; num <= '9'; num++) {                // 试 1-9
                        if (!isValid(board, i, j, num)) continue;
                        board[i][j] = num;
                        if (backtrack(board)) return true;                   // 短路: 找到就停
                        board[i][j] = '.';                                   // 撤
                    }
                    return false;                                            // 当前格 9 数全失败 → 整盘当前状态无解
                }
            }
            return true;                                                     // 所有格非空 → 解出来了
        }
        void solveSudoku(vector<vector<char>>& board) {
            backtrack(board);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def solveSudoku(self, board: list[list[str]]) -> None:
            def is_valid(row: int, col: int, num: str) -> bool:
                for j in range(9):
                    if board[row][j] == num: return False
                for i in range(9):
                    if board[i][col] == num: return False
                box_row, box_col = (row // 3) * 3, (col // 3) * 3            # // 整除, 等价 C++ /
                for i in range(box_row, box_row + 3):
                    for j in range(box_col, box_col + 3):
                        if board[i][j] == num: return False
                return True
            def backtrack() -> bool:
                for i in range(9):
                    for j in range(9):
                        if board[i][j] != '.': continue
                        for num in '123456789':                              # 字符串可迭代, 直接拿到 '1'..'9'
                            if not is_valid(i, j, num): continue
                            board[i][j] = num
                            if backtrack(): return True                      # 短路
                            board[i][j] = '.'
                        return False
                return True
            backtrack()
            # 原地修改 board, 不返回 (题目要求 in-place)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} board
     * @return {void} Do not return anything, modify board in-place instead.
     */
    var solveSudoku = function(board) {
        const isValid = (row, col, num) => {
            for (let j = 0; j < 9; j++) if (board[row][j] === num) return false;
            for (let i = 0; i < 9; i++) if (board[i][col] === num) return false;
            // | 0 等价 Math.floor (正数), 强转 int. 也可写 ((row / 3) | 0) * 3
            const boxRow = Math.floor(row / 3) * 3, boxCol = Math.floor(col / 3) * 3;
            for (let i = boxRow; i < boxRow + 3; i++)
                for (let j = boxCol; j < boxCol + 3; j++)
                    if (board[i][j] === num) return false;
            return true;
        };
        const backtrack = () => {
            for (let i = 0; i < 9; i++) {
                for (let j = 0; j < 9; j++) {
                    if (board[i][j] !== '.') continue;
                    for (let n = 1; n <= 9; n++) {
                        const num = n.toString();                            // char in JS = 长度 1 的 string
                        if (!isValid(i, j, num)) continue;
                        board[i][j] = num;
                        if (backtrack()) return true;                        // 短路
                        board[i][j] = '.';
                    }
                    return false;
                }
            }
            return true;
        };
        backtrack();
    };
    ```

## Complexity

- **Time**: 极难精确, 理论上界 O(9^(空格数)) ≈ 9^81 / 但 isValid 剪掉绝大部分, 实际飞快.
- **Space**: O(空格数) recursion 深度 (≤ 81).

## 易错点

- **递归调用必须 `if (backtrack()) return true;`**: 漏 return → 找到解之后还会被撤销, 一路撤到顶 → 棋盘恢复成空白. 这是 bool 短路模式最容易写错的一行.
- **当前格 9 数字全失败必须 `return false`, 不继续遍历后面格**: 写成 break / 漏 return → 没回退就跑去填后面的格, 后面填出的"解" 跟当前空格冲突. 调试到怀疑人生.

## 相关题目

- [0051. N-Queens](../0051-n-queens/README.md) — 同款棋盘 + 显式回溯 + 局部冲突检查; 输出语义对照 (void vs bool)
- [0046. Permutations](../0046-permutations/README.md) — 同款 `used[]` 思路的源头, 数独的 bitmask 优化就是它的扩展
- [0040. Combination Sum II](../0040-combination-sum-ii/README.md) — 同款显式回溯进入/退出对偶
- 0036\. Valid Sudoku (待补) — 只判合法性不填空, 这题 isValid 的扩展版
- 0079\. Word Search (待补) — 2D 棋盘搜索字符串, 同款 visited + 4 方向 dfs + bool 短路
