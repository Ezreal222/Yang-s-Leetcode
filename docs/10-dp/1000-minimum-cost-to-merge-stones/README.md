# 1000. Minimum Cost to Merge Stones / 合并石头的最低成本

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval, Prefix Sum · 动态规划, 区间, 前缀和
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-to-merge-stones/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `n` 堆石头排成一排. 每次可以把**相邻的 k 堆** 合并成一堆, 代价 = 这 k 堆石头数之和. 求合并成**一堆** 的最小总代价; 无法合成一堆返回 `-1`.

**中文**: `n` 堆石头, 每次合并**相邻 k 堆** (代价 = 这 k 堆总数). 求合成一堆的最小代价, 不行返 `-1`.

## Key Insights

1. **🔑 可行性: `(n - 1) % (k - 1) != 0` 直接返 -1 / Feasibility check**

    每次合并把 k 堆变成 1 堆 → 堆数**减少 `k - 1`**. 从 n 堆合到 1 堆要减 `n - 1`. ⟹ **`(n - 1)` 必须能被 `(k - 1)` 整除**, 否则永远剩不下 1 堆.

    > 这是题目最容易漏的早返. 没这条直接 DP 也能跑, 但浪费时间. **数学约束先判**.

2. **🔑 三维 DP: `dp[i][j][p] = 把 stones[i..j] 合并成恰好 p 堆的最小代价` / 3D state with extra "piles left" axis**

    比 [0312 戳气球](../0312-burst-balloons/README.md) 的二维多一个轴 — 因为区间内"还剩几堆" 是中间状态, 不能直接归约成"已经合完". `p ∈ [1, k]`.

    答案是 `dp[0][n-1][1]` (整段合成 1 堆).

3. **🔑 两类转移 / Two recurrences**

    **(a) 分裂 (p ≥ 2)**: 把 `[i, j]` 切成两段:

    - 左段 `[i, mid]`: 必须合成 **1 堆** (用 `dp[i][mid][1]`)
    - 右段 `[mid+1, j]`: 剩 `p - 1` 堆 (用 `dp[mid+1][j][p-1]`)

    ⟹ `dp[i][j][p] = min(dp[i][mid][1] + dp[mid+1][j][p-1])`, mid 枚举.

    **(b) 合并到 1 堆 (p = 1)**: 当能合成 k 堆时, 再来一次大合并就成 1 堆:

    ⟹ `dp[i][j][1] = dp[i][j][k] + sum(stones[i..j])`

    > "p ≥ 2 的状态" 是中间步, "p = 1 的状态" 是某区间的最终成果. **必须先填 p ≥ 2 再填 p = 1**.

4. **🔑 `mid` 步长是 `k - 1`, 不是 1 / mid steps by k-1, not 1**

    Yang 内层 `for (int mid = i; mid < j; mid += k - 1)`. 为什么不是 `mid++`?

    因为**左段必须能合成 1 堆**, 而能合成 1 堆的区间长度 `L` 必须满足 `(L - 1) % (k - 1) == 0`. 在所有可能的 mid 中, 只有满足这条的才合法 — 这些 mid 在 `i` 之后**每 `k-1` 步** 出现一次. 跳着取就免去判断.

    > 漏写这步, 程序仍会算出"非法切" 的状态 — 它们的 `dp` 是 INT_MAX, 不影响正确性, 但浪费时间. Yang 的优化让复杂度从 O(n⁴) 降到 O(n³).

5. **前缀和 O(1) 求区间和 / Prefix sum for range total**

    "合并到 1 堆" 那一步要加区间总和. 预处理 `prefix[i] = stones[0] + ... + stones[i-1]`, 区间和 = `prefix[j+1] - prefix[i]`. 不做前缀和则每次合并都 O(n) 求和, 整体多一个 n.

6. **遍历顺序: 按区间长度 / Iterate by interval length**

    跟 [0647](../0647-palindromic-substrings/README.md)/[0516](../0516-longest-palindromic-subsequence/README.md) 的"i 倒序 j 顺序" 不同, 这题更适合**按区间长度 len 由小到大**, 因为转移要用到"更短区间的 dp 全部填好". Yang 用的就是 `for (int len = 2; len <= n; len++)`.

7. **复杂度 O(n³ × k) / Complexity**

    - 区间 O(n²)
    - p 维度 O(k)
    - mid 枚举 O(n/(k-1)) = O(n) (有时, 跳 k-1 步)

    总 O(n³ × k / (k-1)) ≈ O(n³). LC `n ≤ 30` 完全够.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int mergeStones(vector<int>& stones, int k) {
            int n = stones.size();
            if ((n - 1) % (k - 1) != 0) return -1;                                // 可行性

            // 前缀和
            vector<int> prefix(n + 1, 0);
            for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + stones[i];

            // dp[i][j][p] = stones[i..j] 合成 p 堆的最低代价
            vector<vector<vector<int>>> dp(n, vector<vector<int>>(n, vector<int>(k + 1, INT_MAX)));
            for (int i = 0; i < n; i++) dp[i][i][1] = 0;                          // 单堆本身就是 1 堆, 代价 0

            for (int len = 2; len <= n; len++) {                                  // 按区间长度
                for (int i = 0; i + len - 1 < n; i++) {
                    int j = i + len - 1;

                    // (a) 分裂转移: p ≥ 2
                    for (int p = 2; p <= k; p++) {
                        for (int mid = i; mid < j; mid += k - 1) {                // ⚠ 步长 k-1
                            if (dp[i][mid][1] != INT_MAX
                                && dp[mid + 1][j][p - 1] != INT_MAX) {
                                dp[i][j][p] = min(dp[i][j][p],
                                                  dp[i][mid][1] + dp[mid + 1][j][p - 1]);
                            }
                        }
                    }

                    // (b) k 堆 → 1 堆 (一次大合并, 代价 = 区间总和)
                    if (dp[i][j][k] != INT_MAX) {
                        dp[i][j][1] = dp[i][j][k] + prefix[j + 1] - prefix[i];
                    }
                }
            }
            return dp[0][n - 1][1];
        }
    };
    ```

=== "Python"
    ```python
    from itertools import accumulate

    class Solution:
        def mergeStones(self, stones: list[int], k: int) -> int:
            n = len(stones)
            if (n - 1) % (k - 1) != 0:
                return -1

            # accumulate 求前缀和, 前面补一个 0 让 prefix[j+1] - prefix[i] 直接是区间和
            # 等价 C++ 的 vector<int>(n+1) + 手写循环
            prefix = [0] + list(accumulate(stones))

            INF = float('inf')
            # dp[i][j][p] = stones[i..j] 合成 p 堆的最低代价
            dp = [[[INF] * (k + 1) for _ in range(n)] for _ in range(n)]
            for i in range(n):
                dp[i][i][1] = 0

            for length in range(2, n + 1):
                for i in range(n - length + 1):
                    j = i + length - 1
                    # (a) p ≥ 2: 分裂
                    for p in range(2, k + 1):
                        for mid in range(i, j, k - 1):                            # 步长 k-1
                            if dp[i][mid][1] < INF and dp[mid + 1][j][p - 1] < INF:
                                dp[i][j][p] = min(dp[i][j][p],
                                                  dp[i][mid][1] + dp[mid + 1][j][p - 1])
                    # (b) k 堆 → 1 堆
                    if dp[i][j][k] < INF:
                        dp[i][j][1] = dp[i][j][k] + prefix[j + 1] - prefix[i]
            return dp[0][n - 1][1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} stones
     * @param {number} k
     * @return {number}
     */
    var mergeStones = function(stones, k) {
        const n = stones.length;
        if ((n - 1) % (k - 1) !== 0) return -1;

        const prefix = new Array(n + 1).fill(0);
        for (let i = 0; i < n; i++) prefix[i + 1] = prefix[i] + stones[i];

        const INF = Infinity;
        // 三维数组: Array.from 嵌套, 防 fill 共享引用
        const dp = Array.from({length: n}, () =>
            Array.from({length: n}, () => new Array(k + 1).fill(INF))
        );
        for (let i = 0; i < n; i++) dp[i][i][1] = 0;

        for (let len = 2; len <= n; len++) {
            for (let i = 0; i + len - 1 < n; i++) {
                const j = i + len - 1;
                for (let p = 2; p <= k; p++) {
                    for (let mid = i; mid < j; mid += k - 1) {
                        if (dp[i][mid][1] < INF && dp[mid + 1][j][p - 1] < INF) {
                            dp[i][j][p] = Math.min(dp[i][j][p],
                                                   dp[i][mid][1] + dp[mid + 1][j][p - 1]);
                        }
                    }
                }
                if (dp[i][j][k] < INF) {
                    dp[i][j][1] = dp[i][j][k] + prefix[j + 1] - prefix[i];
                }
            }
        }
        return dp[0][n - 1][1];
    };
    ```

## Complexity

- **Time**: O(n³).
- **Space**: O(n² × k).

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP 同款"枚举最后操作", 但只需要 2 维状态
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 入门
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 区间 DP 子序列
- 0664\. Strange Printer (待补) — 区间 DP, 同款"分段合并"
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 区间 DP, 同款"枚举切点"
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 区间 DP, min-max 博弈
