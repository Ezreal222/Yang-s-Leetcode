# 0074. Search a 2D Matrix / 搜索二维矩阵

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search, Matrix, Index Mapping · 二分查找, 矩阵, 坐标映射
    - **Link**: [LeetCode](https://leetcode.com/problems/search-a-2d-matrix/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Matrix is globally sorted (row-major)** → treat as flattened array of length `m·n`, **binary search** on `[0, m·n)`; **map `mid → (mid/n, mid%n)`** for cell access.
>
> **中文**: **矩阵行主整体升序** → 视作长 `m·n` 的扁平数组, 在 `[0, m·n)` 上**二分**; **`mid → (mid/n, mid%n)`** 坐标映射取值.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 `m × n` 矩阵, 每行升序 + **每行首 > 上一行末**. 判断 `target` 是否在矩阵中.

**中文**: 行主整体有序的 2D 矩阵找 target.

## Key Insights

1. **🔑 灵魂洞察: 满足条件的矩阵 = 排序数组 (flattened) / Row-major sorted ⇔ flattened sorted array**

    题目性质:

    - **行内升序**.
    - **每行首元素 > 上一行末元素** (关键条件).

    → **按行主序 (row-major) 拉平**得到**严格升序数组**. 二分直接搬来即可.

    ```
    [[1,  3,  5,  7],
     [10, 11, 16, 20],
     [23, 30, 34, 60]]

    Flattened: [1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60] ✅ 排序
    ```

    > **看到"矩阵 + 每行首 > 上行末"** → 反应 flattened 二分. **不需要**两次二分 (先行再列).

2. **🔑 坐标映射: `mid / n = row, mid % n = col` / Index-to-cell mapping**

    扁平索引 mid → 二维坐标:

    - **行**: `mid / n` (整除).
    - **列**: `mid % n`.

    n 是列数. 记住这**灵魂公式** — 数独 (0036 的宫号也是同源) 到矩阵搜索都用它.

    > **一维 ↔ 二维互转公式**: `idx = row × n + col`, `row = idx / n`, `col = idx % n`. 一辈子的工具.

3. **🔑 二分模板: `[l, r]` 闭区间 / Closed interval template**

    ```cpp
    int left = 0, right = m * n - 1;
    while (left <= right) {                          // <= 因为闭区间
        int mid = left + (right - left) / 2;         // 防溢出
        int val = matrix[mid / n][mid % n];
        if (val == target) return true;
        else if (val < target) left = mid + 1;
        else right = mid - 1;
    }
    return false;
    ```

    - **`<=` 不是 `<`**: 闭区间 `[l, r]`, 相等时 mid 是仅剩的元素也要判.
    - **`mid + 1` / `mid - 1`**: 因为 mid 已判过, 缩到不含它的区间.
    - **`left + (right - left) / 2`**: 防 `(l + r) / 2` 溢出 (l, r 都接近 INT_MAX 时).

    > **二分模板选一个 (闭 `[l, r]` 或半开 `[l, r)`) 全程贯彻** — 混用是 bug 温床. Yang 用闭区间.

4. **🔑 避免"两次二分" 反例 / One binary search beats two**

    朴素思路: 先按每行**首元素**二分找目标可能在的行, 再在该行二分找列. **两次二分 O(log m + log n)**.

    **一次二分 O(log(m·n)) = O(log m + log n)** — 复杂度**相同**! 但代码**短一半**, 无边界坑.

    > 面试**首选一次二分**. 两次也对但代码累赘.

5. **🔑 跟 0240 Search a 2D Matrix II 的区别 / vs 0240**

    | | **0074 (本题)** | 0240 Matrix II |
    |---|---|---|
    | 性质 | 行主整体升序 | 仅**每行升 + 每列升** (行首**可能 ≤** 上行末) |
    | 解法 | 一次二分 (flattened) | **阶梯搜索** (右上角开始, `>` 下移, `<` 左移) |
    | 时间 | O(log(m·n)) | O(m + n) |

    > 0074 的强性质让**flattened 二分**成立. 0240 性质弱一档, 只能 O(m + n) 阶梯. **看题目性质选算法**.

6. **🔑 复杂度 O(log(m·n)) 时间, O(1) 空间 / Log-linear time**

    - Time: 二分 log(m × n).
    - Space: 3 个 int.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool searchMatrix(vector<vector<int>>& matrix, int target) {
            int m = matrix.size(), n = matrix[0].size();
            int left = 0, right = m * n - 1;
            while (left <= right) {
                int mid = left + (right - left) / 2;
                int val = matrix[mid / n][mid % n];               // 灵魂映射
                if (val == target) return true;
                else if (val < target) left = mid + 1;
                else right = mid - 1;
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def searchMatrix(self, matrix: list[list[int]], target: int) -> bool:
            m, n = len(matrix), len(matrix[0])
            left, right = 0, m * n - 1
            while left <= right:
                mid = (left + right) // 2       # Python int 无溢出, 直接 //
                # divmod 一步拿 (行, 列); 比 mid // n 和 mid % n 分开更 Pythonic
                r, c = divmod(mid, n)
                val = matrix[r][c]
                if val == target: return True
                elif val < target: left = mid + 1
                else: right = mid - 1
            return False

        # bisect + 手写一维 view 的 pythonic 版 (备选)
        # 用 sortedcontainers 或纯 bisect_left 都行, 这里直接手写更清晰
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} matrix
     * @param {number} target
     * @return {boolean}
     */
    var searchMatrix = function(matrix, target) {
        const m = matrix.length, n = matrix[0].length;
        let left = 0, right = m * n - 1;
        while (left <= right) {
            // JS Number 是 double, 中间 mid 可能是浮点 — 用 Math.floor 强制整数化
            const mid = Math.floor((left + right) / 2);
            // JS 无整除运算符, 用 Math.floor(mid / n) 或位运算 (mid / n) | 0
            const r = Math.floor(mid / n);
            const c = mid % n;
            const val = matrix[r][c];
            if (val === target) return true;
            else if (val < target) left = mid + 1;
            else right = mid - 1;
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(log(m·n)) — 一次二分.
- **Space**: O(1) — 常数指针.

## 相关题目

- [0704. Binary Search](../0704-binary-search/README.md) — 一维二分母题
- [2560. House Robber IV](../2560-house-robber-iv/README.md) — 二分答案 BSA
- [1011. Capacity To Ship Packages Within D Days](../1011-capacity-to-ship-packages-within-d-days/README.md) — BSA + 贪心 check
- [1231. Divide Chocolate](../1231-divide-chocolate/README.md) — BSA
- 0035\. Search Insert Position (待补) — 二分找插入点
- 0240\. Search a 2D Matrix II (待补) — 弱性质版, 阶梯搜索 O(m+n)
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — 矩阵 + 二分答案
- 0668\. Kth Smallest Number in Multiplication Table (待补) — 乘法表 + 二分答案
- 1901\. Find a Peak Element II (待补) — 二维峰值二分
- [0036. Valid Sudoku](../../03-hash-table/0036-valid-sudoku/README.md) — `(r/3)*3 + c/3` 宫号公式, 同族"一维 ↔ 二维互转"
