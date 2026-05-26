# 0375. Guess Number Higher or Lower II / 猜数字大小 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval, Min-Max, Game Theory · 动态规划, 区间, 极小极大, 博弈
    - **Link**: [LeetCode](https://leetcode.com/problems/guess-number-higher-or-lower-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 我心里想一个 `[1, n]` 的数字, 你来猜. 猜对停; 猜错告诉你大了还是小了, 但**你要付 `猜的那个数` 元** (作为惩罚). 求**保证猜中** (最坏情况) 的最少花费.

**中文**: 猜 `[1, n]` 中的数, 错了付"猜的数" 元, 求保证赢 (最坏情况) 的最少钱.

## Key Insights

1. **🔑 区间 DP + 极小极大 (Min-Max) / Interval DP with game-theoretic min-max**

    跟之前区间 DP 不一样, 这道是**博弈题**: 我选猜法 (要 min), 对手 (即"真实答案") 选最坏位置 (要 max). 转移结构是 **`min` 外层 + `max` 内层**.

    > 看到"保证 + 最坏情况" 类问题 → 立刻反应"min-max" 模型. 跟普通"最少代价" 不同.

2. **状态: `dp[i][j] = 答案在 [i, j] 时, 保证猜中的最少花费` / Min guaranteed cost for interval [i, j]**

    "保证" 意味着对所有可能的真实答案, 我们的策略都能在该花费内完成 — 即**最坏情况** 的最少代价.

3. **🔑 转移: 枚举"第一次猜哪个 k" → 子问题选大的那侧 / Enumerate first guess, take max of two sides**

    在 `[i, j]` 内, 枚举**第一次猜的数** `k` (i ≤ k ≤ j). 猜 k 付 `k` 元. 然后:

    - 答案 < k → 落入 `[i, k-1]`, 继续花 `dp[i][k-1]`
    - 答案 > k → 落入 `[k+1, j]`, 继续花 `dp[k+1][j]`

    **对手 (= 真实答案) 会让我落入花费更大的那侧** → 这一步的总花费是 `k + max(dp[i][k-1], dp[k+1][j])`.

    我可以选最优的 k → 对所有 k 取 min:

    $$dp[i][j] = \min_{i \le k \le j} \big( k + \max(dp[i][k-1],\ dp[k+1][j]) \big)$$

    > **`min` 是我能控制的 (选 k), `max` 是对手控制的 (答案在哪)**. 博弈 DP 的灵魂.

4. **🔑 `dp(n + 2)` 哨兵: 防 k±1 越界 / Sentinel sizing to avoid bounds checks**

    Yang 把 dp 开 `(n+2) × (n+2)`, 不是 `(n+1)`. 原因:

    - **下标 0 不用**, 数字从 1 开始. 但 `dp[0][...]` 占位让"前 0 个" 概念存在.
    - **`k = n` 时访问 `dp[k+1][j] = dp[n+1][...]`**, 必须有 `n+1` 行 → dp 开 `n+2`.
    - **`k = i` 时访问 `dp[i][k-1] = dp[i][i-1]`**, 这是"空区间", 默认 0 表示"不用再花" — 自洽.

    > **多开一格 + 默认 0** 是优雅的"边界免特判" 技巧, 跟 [1547 cuts 补哨兵](../1547-minimum-cost-to-cut-a-stick/README.md) 同思想.

5. **`dp[i][j] = INT_MAX` 进入 k 循环前重置 / Reset to INF for min loop**

    跟 [1039](../1039-minimum-score-triangulation-of-polygon/README.md) / [1547](../1547-minimum-cost-to-cut-a-stick/README.md) 一样的小坑: 求 min 必须先放无穷大哨兵, 不然默认 0 一直是最小. **`len ≥ 2` 才进内层**, `len < 2` (空或单元素) 默认 0 表示"不用花钱猜".

6. **为什么不能用二分? / Why not just binary-search guess midpoint?**

    直觉是猜中点最稳, 但本题代价 = **猜的数本身**, 不是步数. 猜数大的位置比猜数小的位置贵 → 中点不一定最优. **必须枚举所有 k 让 DP 找全局最小**.

    > 例: n=4, 猜中点 2: 若答案 > 2, 接下来 [3, 4] 还要再猜, 最坏 +3 = 5. 猜 3 反而更优 (3 + max(dp[1][2], dp[4][4]) = 3 + max(1, 0) = 4). 中点贪心错.

7. **按区间长度遍历 / Iterate by interval length**

    跟所有区间 DP 一样, `for len = 2; len <= n; len++` 保证依赖的更短区间先填好.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int getMoneyAmount(int n) {
            // dp 开 (n+2) × (n+2): 下标 0 不用; +2 让 k+1 = n+1 不越界
            vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
            for (int len = 2; len <= n; len++) {                               // 按长度
                for (int i = 1; i + len - 1 <= n; i++) {
                    int j = i + len - 1;
                    dp[i][j] = INT_MAX;                                        // min 前置 INF
                    for (int k = i; k <= j; k++) {                             // 枚举第一次猜的数
                        // 最坏情况: 对手让答案落在花费大的那侧 → max
                        int worst = max(dp[i][k - 1], dp[k + 1][j]);
                        dp[i][j] = min(dp[i][j], k + worst);                   // 我选最优 k → min
                    }
                }
            }
            return dp[1][n];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def getMoneyAmount(self, n: int) -> int:
            # 同 C++ 哨兵 (n+2), 让 k+1 越界自动取 0
            dp = [[0] * (n + 2) for _ in range(n + 2)]
            for length in range(2, n + 1):
                for i in range(1, n - length + 2):
                    j = i + length - 1
                    # min(...) + 生成器, max 内嵌; 等价 C++ 的 min/max 嵌套
                    dp[i][j] = min(
                        k + max(dp[i][k - 1], dp[k + 1][j])
                        for k in range(i, j + 1)
                    )
            return dp[1][n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var getMoneyAmount = function(n) {
        const dp = Array.from({length: n + 2}, () => new Array(n + 2).fill(0));
        for (let len = 2; len <= n; len++) {
            for (let i = 1; i + len - 1 <= n; i++) {
                const j = i + len - 1;
                let best = Infinity;
                for (let k = i; k <= j; k++) {
                    best = Math.min(best, k + Math.max(dp[i][k - 1], dp[k + 1][j]));
                }
                dp[i][j] = best;
            }
        }
        return dp[1][n];
    };
    ```

## Complexity

- **Time**: O(n³).
- **Space**: O(n²).

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP, 枚举"最后操作"
- [1039. Minimum Score Triangulation of Polygon](../1039-minimum-score-triangulation-of-polygon/README.md) — 同款"枚举中间点"
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 同款"枚举切点"
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 入门
- 0464\. Can I Win (待补) — 博弈 DP, 状压 + 必胜态
- 0486\. Predict the Winner (待补) — 博弈 DP, 选两端
- 0877\. Stone Game (待补) — 博弈 DP, 选两端
