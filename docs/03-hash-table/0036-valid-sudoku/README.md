# 0036. Valid Sudoku / 有效的数独

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Table, Matrix, Bool Array · 哈希表, 矩阵, 布尔数组
    - **Link**: [LeetCode](https://leetcode.com/problems/valid-sudoku/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Every row/col/3×3 box has each digit at most once** → **3 bool tables** `[9][9]` (rows/cols/boxes → digit seen); **box index = `(r/3)*3 + c/3`**; single pass check-and-mark.
>
> **中文**: **每行/列/宫 1-9 不重复** → **3 张 [9][9] bool 表**记录见过; **宫号 = `(r/3)*3 + c/3`**; 一遍扫边查边填.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断 9×9 数独棋盘**当前填的数字有无冲突** (每行 / 每列 / 每个 3×3 宫格 内数字 1-9 不重复). `.` 表示空格, 不检查. **不需要判可解**, 只判"当前状态无冲突".

**中文**: 判数独当前填的数无冲突. 空格 `.` 跳过.

## Key Insights

1. **🔑 哈希轻量版: 3 张 bool 表 / Three bool tables = 3 constant-time hash sets**

    每行 / 列 / 宫都需要一个"数字集". 数字**只 9 种** → `bool[9]` 就是完美的哈希 (O(1) 查, O(1) 插).

    - `rows[r][d]` — 第 r 行是否见过数字 (d+1)
    - `cols[c][d]` — 第 c 列是否见过
    - `boxes[b][d]` — 第 b 宫是否见过

    > 若用 `unordered_set<char>` × 27 张, 也对, 但**多一个哈希跳转开销**. 字符集固定就用数组, 跟 [0242](../0242-valid-anagram/README.md) 同源思想.

2. **🔑 宫号公式: `(r/3)*3 + c/3` / The magic box index**

    9 个宫格编号 0..8. 给 `(r, c)` 算它属于哪个宫:

    ```
    r / 3 ∈ {0, 1, 2}          ← 宫的"行块"
    c / 3 ∈ {0, 1, 2}          ← 宫的"列块"
    box_id = (r/3) × 3 + (c/3) ∈ 0..8
    ```

    展开就是**行块 × 3 + 列块** — 二维压一维的经典手法. 记住这个公式, 数独类题**必考**.

    ```
    宫号布局:
    0 1 2
    3 4 5
    6 7 8
    ```

3. **🔑 一遍扫 vs 三遍扫 / One pass vs three passes**

    另一种写法是**三遍**: 先扫每行, 再扫每列, 再扫每宫. 逻辑清晰, 但 3× 时间常数.

    Yang 的**一遍扫**同时维护 3 张表, 每格**只访问一次**, 每次判 3 张表. 常数更好, 代码也短.

    > **"多个约束同时验证" → 单趟 + 多张状态表** 是通用的优化模板. `if (rowsA[i] || rowsB[i] || ...) return false;` 短路.

4. **🔑 一发现冲突就短路 return false / Early return on conflict**

    Yang 的核心一段:

    ```cpp
    if (rows[r][d] || cols[c][d] || boxes[b][d]) return false;
    rows[r][d] = cols[c][d] = boxes[b][d] = true;
    ```

    **先查后填**: 若任一表已见过, 说明重复, 立刻返 false. 否则 3 张表都标"见过". `||` 短路 → 最多查 3 次.

5. **🔑 空格 `.` 跳过, 不需要"合法性" / Skip `.`, no solvability check**

    题目**只要 "当前无冲突"** — 允许空格, 允许数独当前解不出来 (只要没直接矛盾). 遇 `.` `continue` 即可, 不进任何表.

    > **别过度设计** — 有人会想"要不要检查最终能否解出?" → 不用. LeetCode 题面严格划分了范围.

6. **🔑 位掩码优化 (工程向) / Bitmask optimization**

    3 × 9 = 27 个 bool 数组 → 可以压成 27 个 int (每个 int 用 9 bit): `rows[r]` 是一个 int, 第 d bit 表示见过数字 d+1.

    ```cpp
    if ((rows[r] >> d) & 1) return false;
    rows[r] |= (1 << d);
    ```

    内存从 27 × 9 = 243 字节压到 27 × 4 = 108 字节, 也省了 cache miss. 代码短一点但读起来更"密". Yang 的 bool 版**教学更清晰**, 位掩码是工程 preference.

7. **复杂度 O(1) 时间, O(1) 空间 / Constant**

    棋盘固定 9×9 = 81 格 → 81 次迭代. 3 张 9×9 = 243 bool 常量空间. **对**这题来说都是 O(1) — 因为规模固定.

    真要用 n 表达: O(n²) 时间, O(n²) 空间 for n×n 数独.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isValidSudoku(vector<vector<char>>& board) {
            bool rows[9][9] = {false};                           // rows[r][d] = 行 r 见过数字 d+1
            bool cols[9][9] = {false};
            bool boxes[9][9] = {false};

            for (int r = 0; r < 9; r++) {
                for (int c = 0; c < 9; c++) {
                    char ch = board[r][c];
                    if (ch == '.') continue;                     // 空格跳过
                    int d = ch - '1';                            // 数字 → 0..8 下标
                    int b = (r / 3) * 3 + (c / 3);               // 宫号公式

                    if (rows[r][d] || cols[c][d] || boxes[b][d]) return false;

                    rows[r][d] = cols[c][d] = boxes[b][d] = true;
                }
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isValidSudoku(self, board: list[list[str]]) -> bool:
            # 3 张 set 的 list. 每个 set 就是"该行/列/宫已见的数字集".
            # C++ 用 bool[9][9] 是因为数字固定 9 种; Python set 更直观, 语义 = "已见集合"
            # 27 张 set 一次分配, [set() for _ in range(9)] 必须用推导式 (不能用 [set()] * 9, 会共享引用)
            rows = [set() for _ in range(9)]
            cols = [set() for _ in range(9)]
            boxes = [set() for _ in range(9)]

            for r in range(9):
                for c in range(9):
                    ch = board[r][c]
                    if ch == '.':
                        continue
                    b = (r // 3) * 3 + (c // 3)     # // 整除, 等价 C++ r / 3
                    # in / not in 对 set 是 O(1), 跟 C++ bool[d] 检查等价
                    if ch in rows[r] or ch in cols[c] or ch in boxes[b]:
                        return False
                    rows[r].add(ch); cols[c].add(ch); boxes[b].add(ch)
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} board
     * @return {boolean}
     */
    var isValidSudoku = function(board) {
        // JS 用 Set. Array.from 工厂函数避免共享引用坑 (跟 Python 的 [set() for _ in ...] 同理)
        // 不能 new Array(9).fill(new Set()) — 那样所有槽指向同一个 Set
        const rows = Array.from({length: 9}, () => new Set());
        const cols = Array.from({length: 9}, () => new Set());
        const boxes = Array.from({length: 9}, () => new Set());

        for (let r = 0; r < 9; r++) {
            for (let c = 0; c < 9; c++) {
                const ch = board[r][c];
                if (ch === '.') continue;
                // Math.floor 相当于 C++ 的整数除法. JS 的 / 是浮点除
                const b = Math.floor(r / 3) * 3 + Math.floor(c / 3);
                // Set.prototype.has 是 O(1) 平均, 跟 C++ bool 数组等价效率
                if (rows[r].has(ch) || cols[c].has(ch) || boxes[b].has(ch)) return false;
                rows[r].add(ch); cols[c].add(ch); boxes[b].add(ch);
            }
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(1) — 固定 81 格 (通用 n² 数独: O(n²)).
- **Space**: O(1) — 27 张固定 9-bool 表 (通用: O(n²)).

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数/bool 数组作哈希 母题
- 0037\. Sudoku Solver (待补) — 本题延伸: **回溯**填数独, 用相同的 rows/cols/boxes 状态表剪枝
- 0079\. Word Search (待补) — 网格 + 回溯 (状态另建)
- 0289\. Game of Life (待补) — 网格状态判定, 位掩码写法参考
- 0999\. Available Captures for Rook (待补) — 8×8 棋盘遍历
- 0073\. Set Matrix Zeroes (待补) — 矩阵原地标记
