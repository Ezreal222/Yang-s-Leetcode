# 0085. Maximal Rectangle / 最大矩形

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Monotonic Stack, Matrix, DP · 单调栈, 矩阵, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/maximal-rectangle/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 `0/1` 二维矩阵, 求**只含 `1`** 的最大矩形面积.

**中文**: `0/1` 矩阵, 求全 `1` 子矩形的最大面积.

## Key Insights

1. **🔑 二维 → 一维: 把每行当 histogram 跑 [0084](../0084-largest-rectangle-in-histogram/README.md) / Reduce to histogram per row**

    把矩阵想成"每行是一个柱状图的底" — 每列的高度 = **从该行往上连续 `1` 的个数**. 对每一行跑一次 [0084 柱状图最大矩形](../0084-largest-rectangle-in-histogram/README.md), 取所有行的最大值.

    > **二维问题 = 一维子问题 × m 次** 是矩阵题常见降维手法. 跟 0407 接雨水 II → 最小堆, 0221 最大正方形 → 二维 DP 同精神 — **找一条主线把二维拍扁**.

2. **🔑 高度数组的递推: 当前列是 1 就累加, 0 就清零 / Heights array recurrence**

    扫第 i 行时:

    $$heights[j] = \begin{cases} heights[j] + 1, & matrix[i][j] = 1 \\ 0, & matrix[i][j] = 0 \end{cases}$$

    `heights[j]` 表示"以第 i 行第 j 列为底, 向上能伸多高的全 1 柱子". 一旦遇到 0, 柱子断了, 清零.

    > 这是把"二维历史" 压成"一维快照" — heights 数组**滚动更新**, O(n) 空间.

3. **状态: `heights[j]` 在每行结束时是当前行的 histogram / Snapshot per row**

    每行处理完, heights 描述了"以这行为底" 的柱状图. 调用 0084 算这行的答案. **`maxArea` 跨行取 max**.

4. **复杂度 O(m × n) — 跟矩阵总元素数同阶 / Linear in input size**

    每行 O(n) 更新 heights, O(n) 跑 0084. m 行总 O(m × n). 跟读入矩阵同阶, 已是理论下界.

5. **⚠ 代码小细节: `largestRectangleArea` 的 `push_back(0)` 改了外部 heights / Side effect**

    Yang 调 `largestRectangleArea(heights)` 时, 函数内 `heights.push_back(0)` **修改了外部的 heights**! 每次循环 heights 多一个 0, 长到 n+m+1. 外层 `for j < n` 只看前 n 个, 答案仍**正确**, 但效率有损 (内层多扫几个无用 0).

    干净写法之一: 函数末尾 `heights.pop_back()` 还原; 或参数改成值传递 `vector<int>` (拷贝, 但内层独立). Yang 当前版能过, 不过算 code smell.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maximalRectangle(vector<vector<char>>& matrix) {
            if (matrix.empty()) return 0;
            int m = matrix.size(), n = matrix[0].size();
            vector<int> heights(n, 0);                             // 滚动 heights, O(n)
            int maxArea = 0;
            for (int i = 0; i < m; i++) {
                // 每列递推: 是 '1' 累加, 是 '0' 清零
                for (int j = 0; j < n; j++) {
                    heights[j] = (matrix[i][j] == '1') ? heights[j] + 1 : 0;
                }
                maxArea = max(maxArea, largestRectangleArea(heights));
            }
            return maxArea;
        }

    private:
        int largestRectangleArea(vector<int>& heights) {
            heights.push_back(0);                                  // 末尾哨兵
            stack<int> stk;
            int maxArea = 0;
            for (int i = 0; i < (int)heights.size(); i++) {
                while (!stk.empty() && heights[i] < heights[stk.top()]) {
                    int h = heights[stk.top()]; stk.pop();
                    int left = stk.empty() ? -1 : stk.top();
                    maxArea = max(maxArea, h * (i - left - 1));
                }
                stk.push(i);
            }
            heights.pop_back();                                    // ✓ 还原, 避免副作用累积
            return maxArea;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maximalRectangle(self, matrix: list[list[str]]) -> int:
            if not matrix:
                return 0
            m, n = len(matrix), len(matrix[0])
            heights = [0] * n
            max_area = 0

            def largest_rectangle(h):
                # 复制一份避免污染外部 heights — Pythonic
                stk = []
                best = 0
                # 末尾接 0 当哨兵, 不修改原列表
                for i, v in enumerate(h + [0]):
                    while stk and v < h_with_sentinel[stk[-1]]:
                        top_h = h_with_sentinel[stk.pop()]
                        left = stk[-1] if stk else -1
                        best = max(best, top_h * (i - left - 1))
                    stk.append(i)
                return best

            # 简洁版: 用切片复制 + 直接传函数
            h_with_sentinel = None  # 占位, 看下面更干净写法
            for i in range(m):
                for j in range(n):
                    heights[j] = heights[j] + 1 if matrix[i][j] == '1' else 0
                h_with_sentinel = heights + [0]  # 临时副本带哨兵
                stk = []
                for i2, v in enumerate(h_with_sentinel):
                    while stk and v < h_with_sentinel[stk[-1]]:
                        top_h = h_with_sentinel[stk.pop()]
                        left = stk[-1] if stk else -1
                        max_area = max(max_area, top_h * (i2 - left - 1))
                    stk.append(i2)
            return max_area
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} matrix
     * @return {number}
     */
    var maximalRectangle = function(matrix) {
        if (!matrix.length) return 0;
        const m = matrix.length, n = matrix[0].length;
        const heights = new Array(n).fill(0);
        let maxArea = 0;

        const largestRectangle = (h) => {
            const arr = [...h, 0];                                 // spread 复制 + 哨兵, 不污染
            const stk = [];
            let best = 0;
            for (let i = 0; i < arr.length; i++) {
                while (stk.length && arr[i] < arr[stk[stk.length - 1]]) {
                    const top = stk.pop();
                    const left = stk.length ? stk[stk.length - 1] : -1;
                    best = Math.max(best, arr[top] * (i - left - 1));
                }
                stk.push(i);
            }
            return best;
        };

        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                heights[j] = matrix[i][j] === '1' ? heights[j] + 1 : 0;
            }
            maxArea = Math.max(maxArea, largestRectangle(heights));
        }
        return maxArea;
    };
    ```

## Complexity

- **Time**: O(m × n) — m 行 × O(n) histogram.
- **Space**: O(n) — heights 数组 + 栈.

## 相关题目

- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — **本题的一维子问题**, 必先掌握
- [0042. Trapping Rain Water](../0042-trapping-rain-water/README.md) — 单调栈经典 Hard
- 0221\. Maximal Square (待补) — 同款"全 1 子矩形" 但求正方形, 用二维 DP 更优
- [1504. Count Submatrices With All Ones](../1504-count-submatrices-with-all-ones/README.md) — 同款 histogram 思想, 求**子矩形个数** 而非最大面积
- 1727\. Largest Submatrix With Rearrangements (待补) — 0085 进阶, 允许列重排
- 0407\. Trapping Rain Water II (待补) — 二维接雨水, 用最小堆
