# 1504. Count Submatrices With All Ones / 统计全 1 子矩形

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Matrix, Histogram · 单调栈, 矩阵, 柱状图
    - **Link**: [LeetCode](https://leetcode.com/problems/count-submatrices-with-all-ones/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `0/1` 矩阵, 统计**全为 `1`** 的子矩形 (任意大小) 的**个数**.

**中文**: 0/1 矩阵, 求全 1 子矩形的总数.

## Key Insights

1. **🔑 二维 → 一维降维 (跟 [0085 最大矩形](../0085-maximal-rectangle/README.md) 同思路) / Reduce to histogram per row**

    把每行当作"柱状图的底", `heights[j]` = 第 j 列向上连续 `1` 的个数 (跟 0085 一字不差). 然后**对每行求"以该行为底的全 1 子矩形个数"**, 跨行累加.

    > **0085 求最大矩形面积** vs **1504 求矩形个数** — 同 histogram 结构, 输出不同.

2. **🔑 计数避免重复: 按"底-右下角" 唯一归属 / Each rectangle counted by its bottom-right corner**

    扫到行 i 时, 我们数的是**底边在第 i 行的所有全 1 子矩形**. 每个子矩形有唯一的"最底行" (它的底边), 跨行累加自然不重不漏.

    > 同款"按某个唯一特征点归属" 思维: [0042 接雨水](../0042-trapping-rain-water/README.md) 按"当前柱子接的水", [0084 最大矩形](../0084-largest-rectangle-in-histogram/README.md) 按"以该柱为最矮".

3. **🔑 行内枚举: 对每个右边界 j, 倒着扫左边界 k, 维护 minH / Enumerate right, track min as left shrinks**

    固定行 i, 固定右边界 j. 左边界 k 从 j 到 0:

    - `minH = min(heights[k..j])` (滚动维护, 每次取 `min(minH_old, heights[k])`)
    - 这一对 (k, j) 贡献 `minH` 个子矩形 — **它们高度可以是 `1, 2, ..., minH`**, 共 minH 个.

    累加所有 (j, k) 对. **行内总贡献 = `Σ_j Σ_{k≤j} min(heights[k..j])`**.

    > 这正是"子数组最小值之和" (Sum of Subarray Minimums) 的二维形式 — 跟 0907 一一对应.

4. **复杂度 O(m × n²) — 朴素枚举 / Naive enumeration**

    每行 O(n²) 双循环 (j × k). m 行总 O(m × n²). LC 约束 `m, n ≤ 150` → 3.4M, 完全可以.

    > Yang 的做法就是这个朴素版, 代码短易读, 在 LC 数据下够用.

5. **🔑 进阶: O(m × n) 单调栈 (Sum of Subarray Minimums) / Monotonic stack optimization**

    把内层"对每个 j 倒扫 k 求 min" 优化掉. 对每行的 heights 数组, 用单调栈计算"以每个位置为右端点的所有子数组最小值之和" — 这是 0907 Sum of Subarray Minimums (待补) 的经典套路. 行内 O(n).

    本题如果出 `m, n ≤ 10^4` 就必须上这个优化. LC 当前数据 O(m·n²) 也能过.

    > **递增**: 朴素 O(n²) → 单调栈 O(n) — 跟 [0084](../0084-largest-rectangle-in-histogram/README.md) 把柱状图最大矩形从 O(n²) 降到 O(n) 是同一类降阶.

## Solution

=== "C++"
    === "v1 (Yang 原版): 朴素 O(m × n²)"
        ```cpp
        class Solution {
        public:
            int numSubmat(vector<vector<int>>& mat) {
                int m = mat.size(), n = mat[0].size();
                vector<int> heights(n, 0);
                int total = 0;
                for (int i = 0; i < m; i++) {
                    // 更新当前行的 heights (跟 0085 一字不差)
                    for (int j = 0; j < n; j++) {
                        heights[j] = (mat[i][j] == 1) ? heights[j] + 1 : 0;
                    }
                    // 对每个右边界 j, 倒扫左边界 k, 维护 minH 累加
                    for (int j = 0; j < n; j++) {
                        int minH = heights[j];
                        for (int k = j; k >= 0; k--) {
                            minH = min(minH, heights[k]);
                            total += minH;                         // (k, j) 贡献 minH 个矩形
                        }
                    }
                }
                return total;
            }
        };
        ```

    === "v2: O(m × n) 单调栈 (进阶)"
        ```cpp
        // 用单调栈算"以每个位置为右端点的所有子数组最小值之和"
        class Solution {
        public:
            int numSubmat(vector<vector<int>>& mat) {
                int m = mat.size(), n = mat[0].size();
                vector<int> heights(n, 0);
                int total = 0;
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        heights[j] = (mat[i][j] == 1) ? heights[j] + 1 : 0;
                    }
                    // sum[j] = 以 j 为右端点的所有子数组最小值之和
                    vector<int> sum(n, 0);
                    stack<int> stk;                                // 单调递增栈 (存索引)
                    for (int j = 0; j < n; j++) {
                        // 弹出 ≥ heights[j] 的栈顶, 它们的"被 j 取代为最小" 的贡献要重算
                        while (!stk.empty() && heights[stk.top()] >= heights[j]) stk.pop();
                        if (stk.empty()) {
                            sum[j] = heights[j] * (j + 1);
                        } else {
                            int prev = stk.top();
                            sum[j] = sum[prev] + heights[j] * (j - prev);
                        }
                        stk.push(j);
                        total += sum[j];
                    }
                }
                return total;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def numSubmat(self, mat: list[list[int]]) -> int:
            m, n = len(mat), len(mat[0])
            heights = [0] * n
            total = 0
            for i in range(m):
                # 更新 heights
                for j in range(n):
                    heights[j] = heights[j] + 1 if mat[i][j] == 1 else 0
                # 对每个右边界 j, 倒扫 k, 累加 minH
                for j in range(n):
                    min_h = heights[j]
                    for k in range(j, -1, -1):
                        if heights[k] < min_h:
                            min_h = heights[k]
                        total += min_h
            return total
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} mat
     * @return {number}
     */
    var numSubmat = function(mat) {
        const m = mat.length, n = mat[0].length;
        const heights = new Array(n).fill(0);
        let total = 0;
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                heights[j] = mat[i][j] === 1 ? heights[j] + 1 : 0;
            }
            for (let j = 0; j < n; j++) {
                let minH = heights[j];
                for (let k = j; k >= 0; k--) {
                    if (heights[k] < minH) minH = heights[k];
                    total += minH;
                }
            }
        }
        return total;
    };
    ```

## Complexity

- **Time**: O(m × n²) (v1) / O(m × n) (v2 单调栈).
- **Space**: O(n) — heights 数组 (v1) / O(n) heights + sum + 栈 (v2).

## 相关题目

- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 一维 histogram 最大矩形
- [0085. Maximal Rectangle](../0085-maximal-rectangle/README.md) — 同款"二维 → histogram 降维", 求**最大** 而非**个数**
- [0907. Sum of Subarray Minimums](../0907-sum-of-subarray-minimums/README.md) — v2 的核心子问题, 单调栈算每个位置作最小的贡献
- 0221\. Maximal Square (待补) — 求最大全 1 正方形, 二维 DP 更优
- 1727\. Largest Submatrix With Rearrangements (待补) — 0085 进阶, 允许列重排
