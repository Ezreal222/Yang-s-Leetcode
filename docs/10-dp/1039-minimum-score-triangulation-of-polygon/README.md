# 1039. Minimum Score Triangulation of Polygon / 多边形三角剖分的最低得分

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval · 动态规划, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 凸 n 边形顶点按顺序标 `values[0..n-1]`. 把它**三角剖分** 成 n-2 个三角形 (画 n-3 条不交叉的对角线). 每个三角形得分 = 三顶点值乘积. **求所有三角形得分总和的最小值**.

**中文**: 凸 n 边形三角剖分 (n-2 个三角形), 每个三角形得分 = 三顶点值乘积, 求总分最小.

## Key Insights

1. **🔑 跟 [0312 戳气球](../0312-burst-balloons/README.md) 是结构孪生 / Structural twin of 0312**

    两题的转移**完全同形态**: 三层循环, 枚举中间点 k 把区间切成两段, 每段独立 DP, 加上"含 k 的关键步骤" 贡献.

    | | [0312 戳气球](../0312-burst-balloons/README.md) | **1039 (本题)** |
    |---|---|---|
    | 题意 | 最后戳 k, 拿 `nums[i]×nums[k]×nums[j]` 金币 (i, j 为残余邻居) | 用三角形 `(i, k, j)`, 得分 `values[i]×values[k]×values[j]` |
    | DP 结构 | 求 max | 求 min |
    | 哨兵 | **要** (两端补 1) | **不要** (顶点本身就是边界) |
    | 子问题 | `dp[i][k] + dp[k][j] + ...` | `dp[i][k] + dp[k][j] + ...` |

    > **看到"区间内枚举一个点切两段, 每段子问题独立, 加上含该点的贡献"** → 立刻反应"0312-同款" 套路.

2. **状态: `dp[i][j] = 以顶点 i, i+1, ..., j 构成的 (子) 多边形三角剖分最小得分` / Sub-polygon by consecutive vertices**

    把 "顶点 `i`, `i+1`, ..., `j`" + "**边 `(i, j)`**" 当成一个子多边形 (注意是连续顶点 + 一条闭合边). 把它三角剖分的最小总分就是 `dp[i][j]`.

    答案是 `dp[0][n-1]` (整个 n 边形).

3. **🔑 转移: 枚举包含边 (i, j) 的那个三角形的第三顶点 / Enumerate the apex of the triangle containing edge (i, j)**

    边 `(i, j)` 必属于**某个三角形** `(i, k, j)`, 其中 `k` 是 i+1..j-1 中的某点. 选定 k 后, 多边形被这个三角形切成三块:

    - 三角形 `(i, k, j)` 自己 — 得分 `values[i] × values[k] × values[j]`
    - 左子多边形 `i, i+1, ..., k` — 最小得分 `dp[i][k]`
    - 右子多边形 `k, k+1, ..., j` — 最小得分 `dp[k][j]`

    取所有 k 的最小:

    $$dp[i][j] = \min_{i < k < j} \big( dp[i][k] + dp[k][j] + values[i] \times values[k] \times values[j] \big)$$

    > 跟 0312 几乎一字不差, 但语义不同 — 这里 k 是"三角形顶点", 那里 k 是"最后戳的气球".

4. **边界: `j - i < 2` (顶点 < 3) → dp = 0 / Too few vertices, no triangle**

    Yang 的 `for (int j = i + 2; j < n; j++)` 跳过了 `j == i` 和 `j == i + 1` (1 或 2 个顶点). 默认值 0 正好.

    `j - i == 2` 时只有 3 个顶点, 形成唯一一个三角形, `dp = values[i] * values[i+1] * values[j]` — 由 `k = i+1` 时的转移自然算出, 不用特判.

5. **遍历顺序: i 倒序 j 顺序 — 跟所有区间 DP 一致 / Same as 0647 / 0516 / 0312**

    `dp[i][j]` 依赖 `dp[i][k]` (j 更小) 和 `dp[k][j]` (i 更大), 必须先于 `dp[i][j]` 算好.

6. **`dp[i][j] = INT_MAX` 必须在每个 (i, j) 进入内层前重置 / Reset to INF before min loop**

    Yang 写 `dp[i][j] = INT_MAX;` 在 `for k` 之前 — 必需. 默认 0 是空多边形的初值, 进入"要找 min" 的循环前要先设无穷大, 让 min 能更新.

    > 这是计数/求最值 DP 的常见小坑: **求 min 必须先放 INF 哨兵**, 不像计数那样默认 0 就行.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minScoreTriangulation(vector<int>& values) {
            int n = values.size();
            vector<vector<int>> dp(n, vector<int>(n, 0));          // < 3 顶点的子多边形得分 0
            for (int i = n - 1; i >= 0; i--) {                     // i 倒序
                for (int j = i + 2; j < n; j++) {                  // j ≥ i + 2 才有内部
                    dp[i][j] = INT_MAX;                            // 先置哨兵, 再取 min
                    for (int k = i + 1; k < j; k++) {              // k 是含边 (i,j) 的三角形第三顶点
                        dp[i][j] = min(dp[i][j],
                                       dp[i][k] + dp[k][j]
                                       + values[i] * values[k] * values[j]);
                    }
                }
            }
            return dp[0][n - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minScoreTriangulation(self, values: list[int]) -> int:
            n = len(values)
            # < 3 顶点子多边形默认 0, 跟 j ≥ i + 2 的循环边界自洽
            dp = [[0] * n for _ in range(n)]
            for i in range(n - 1, -1, -1):
                for j in range(i + 2, n):
                    dp[i][j] = float('inf')                        # 求 min 先置哨兵
                    for k in range(i + 1, j):
                        gain = values[i] * values[k] * values[j]
                        dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + gain)
            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} values
     * @return {number}
     */
    var minScoreTriangulation = function(values) {
        const n = values.length;
        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        for (let i = n - 1; i >= 0; i--) {
            for (let j = i + 2; j < n; j++) {
                dp[i][j] = Infinity;
                for (let k = i + 1; k < j; k++) {
                    const gain = values[i] * values[k] * values[j];
                    dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + gain);
                }
            }
        }
        return dp[0][n - 1];
    };
    ```

## Complexity

- **Time**: O(n³).
- **Space**: O(n²).

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — **结构孪生**, max 版本 + 哨兵
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 入门
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 区间 DP 子序列
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 区间 DP 进阶, 3D 状态
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 区间 DP, 同款"枚举切点 / 顶点"
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 区间 DP, min-max 博弈
