# 0312. Burst Balloons / 戳气球

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval · 动态规划, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/burst-balloons/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 n 个气球价值 `nums[i]`. 戳破气球 i 得 `nums[left] × nums[i] × nums[right]` 金币 (`left`, `right` 是当前剩余气球里的左右邻居; 边界外当 1). 求**戳完所有气球的最大总金币**.

**中文**: 戳气球 i 得 `左邻 × 当前 × 右邻` 金币 (边界外算 1). 求戳完所有气球的最大总金币.

## Key Insights

1. **🔑 区间 DP 进阶: 比 [0647](../0647-palindromic-substrings/README.md)/[0516](../0516-longest-palindromic-subsequence/README.md) 难得多 / Advanced interval DP**

    回文 DP 转移是"看两端" — 简单. 本题转移要枚举**最后一个戳破的气球**, 比基础区间 DP 难一档. 是经典面试 Hard, 跟编辑距离同级.

2. **🔑 反向思维: 枚举"最后破" 而不是"最先破" / Last burst, not first**

    新手直觉是"枚举先戳哪个" — 但戳完后剩余气球的相邻关系改变, 子问题不独立, 没法 DP.

    **倒着想**: 在区间内, 哪个气球**最后戳破** `k`? 当 k 是最后一个破的:

    - k 左边的全部已经破完 (那是子问题 `(i, k)`)
    - k 右边的全部已经破完 (那是子问题 `(k, j)`)
    - **k 最后破时, 它的邻居就是 `i` 和 `j`** (因为中间气球都没了, 只剩边界 i, j 和 k 自己)
    - 戳 k 得 `nums[i] × nums[k] × nums[j]`

    左右两段独立 + 当前 k 的收益, 子问题终于**独立**.

    > **"最后" 视角让子问题独立** — 这是这道题的灵魂. 区间 DP 里"最后一步" 不一定是空间上的末尾, 可以是时间上的最后操作.

3. **🔑 哨兵: 两端补 1 / Sentinel 1s on both sides**

    把 `nums` 包装成 `balloons = [1, nums[0], ..., nums[n-1], 1]`, 长度 `m = n + 2`. 边界外的"虚拟邻居" 直接当作 `balloons[0] = balloons[m-1] = 1` 用. 转移公式里不用单独判左/右边界.

    > 哨兵技巧让代码 short and clean. 跟 [0718](../0718-maximum-length-of-repeated-subarray/README.md) 的 `(n+1) × (m+1)` 哨兵同思路.

4. **状态: `dp[i][j] = 开区间 (i, j) 全部戳破能拿的最大金币` / Open interval (i, j)**

    `i, j` 是**未被戳的左右边界**, **不在区间内** (开区间). 区间内的气球都要戳完. 答案是 `dp[0][m-1]` (整段 + 两哨兵).

5. **转移: 枚举 k 当"最后戳" / Transition: enumerate which is last burst**

    $$dp[i][j] = \max_{i < k < j} \big( dp[i][k] + dp[k][j] + balloons[i] \times balloons[k] \times balloons[j] \big)$$

    需要 `j - i ≥ 2` (区间内至少要有一个气球). 否则空区间, `dp[i][j] = 0` (默认值正好).

6. **遍历顺序: i 倒序, j 顺序, k 在内 / Same as 0647/0516, plus inner k**

    `dp[i][j]` 依赖 `dp[i][k]` (i 同, k 更小) 和 `dp[k][j]` (k 更大, j 同). 既要 i 后于更大的 i, 又要 j 先于更小的 j → **外 i 倒序, 中 j 顺序, 内 k 枚举**.

    > 三层循环 O(n³). 数据规模 n ≤ 500, 1.25 × 10^8 操作, 卡时限但能过.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxCoins(vector<int>& nums) {
            int n = nums.size();
            // 两端补 1, 简化边界
            vector<int> balloons(n + 2, 1);
            for (int i = 0; i < n; i++) balloons[i + 1] = nums[i];
            int m = n + 2;

            vector<vector<int>> dp(m, vector<int>(m, 0));
            // i 倒序, j 顺序, k 在 (i, j) 内
            for (int i = m - 1; i >= 0; i--) {
                for (int j = i + 2; j < m; j++) {                  // j - i ≥ 2 才有内部气球
                    for (int k = i + 1; k < j; k++) {              // k 是"最后戳" 的气球
                        dp[i][j] = max(dp[i][j],
                                       dp[i][k] + dp[k][j]
                                       + balloons[i] * balloons[k] * balloons[j]);
                    }
                }
            }
            return dp[0][m - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxCoins(self, nums: list[int]) -> int:
            # 两端补 1
            balloons = [1] + nums + [1]
            m = len(balloons)
            dp = [[0] * m for _ in range(m)]

            # i 倒序, j 顺序, k 内枚举"最后戳"
            for i in range(m - 1, -1, -1):
                for j in range(i + 2, m):
                    for k in range(i + 1, j):
                        # k 最后戳时, 邻居就是 i, j (中间已全空)
                        gain = balloons[i] * balloons[k] * balloons[j]
                        dp[i][j] = max(dp[i][j], dp[i][k] + dp[k][j] + gain)
            return dp[0][m - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var maxCoins = function(nums) {
        const balloons = [1, ...nums, 1];                          // spread + 两哨兵
        const m = balloons.length;
        const dp = Array.from({length: m}, () => new Array(m).fill(0));
        for (let i = m - 1; i >= 0; i--) {
            for (let j = i + 2; j < m; j++) {
                for (let k = i + 1; k < j; k++) {
                    const gain = balloons[i] * balloons[k] * balloons[j];
                    dp[i][j] = Math.max(dp[i][j], dp[i][k] + dp[k][j] + gain);
                }
            }
        }
        return dp[0][m - 1];
    };
    ```

## Complexity

- **Time**: O(n³).
- **Space**: O(n²).

## 相关题目

- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 入门
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 区间 DP 子序列版
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 同款区间 DP + min-max 博弈
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 同款"枚举切点", 区间 DP + 哨兵 + 排序
- 0664\. Strange Printer (待补) — 同款区间合并 DP
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 区间 DP 进阶, 三维状态 + 可行性约束
