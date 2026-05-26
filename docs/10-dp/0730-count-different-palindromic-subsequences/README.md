# 0730. Count Different Palindromic Subsequences / 统计不同回文子序列

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval, String, Counting, Inclusion-Exclusion · 动态规划, 区间, 字符串, 计数, 容斥
    - **Link**: [LeetCode](https://leetcode.com/problems/count-different-palindromic-subsequences/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s` (只含 `a-d`). 求 `s` 中**不同回文子序列** 的数量, 模 `10^9 + 7`. **"不同"** 按字符串值算, 同一回文出现多次只计一次.

**中文**: 求 `s` 的**不同 (distinct)** 回文子序列数, 模 1e9+7. 同值的回文只算一次.

## Key Insights

1. **🔑 难点是"不同" (去重), 不是"个数" / The hard part is distinct, not count**

    跟 [0647 回文子串计数](../0647-palindromic-substrings/README.md) (按位置区分, 同值不同位置算两个) **本质不同**. 本题要按**字符串值** 去重, 转移要做 **容斥**.

    > 看到"distinct" + 子序列 → 立刻警觉, 简单 DP 会重复计数. 必须用容斥或显式去重.

2. **状态: `dp[i][j] = s[i..j] 中不同回文子序列的数量` / Distinct count in interval**

    跟所有区间 DP 一样, `dp[i][i] = 1` (单字符自身就是一种).

3. **🔑 转移分两大情况 / Two big cases**

    **情况 A: `s[i] != s[j]`** — 用容斥 (Inclusion-Exclusion):

    $$dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]$$

    - `dp[i+1][j]`: 不含 `s[i]` 的所有回文子序列
    - `dp[i][j-1]`: 不含 `s[j]` 的所有回文子序列
    - `dp[i+1][j-1]`: 两边都不含 — 被重复计了一次, 减掉

    > 标准二维容斥. 跟概率论 `P(A∪B) = P(A) + P(B) - P(A∩B)` 完全同型.

4. **🔑 情况 B: `s[i] == s[j]`** — 找内部相同字符做三分支 / Three subcases by middle occurrences

    设 `lo` = `s[i+1..j-1]` 里**最左** 一个 `s[i]` 的位置, `hi` = **最右** 一个. 三种:

    | 内部 s[i] 出现 | 转移 | 解释 |
    |---|---|---|
    | **0 次** (`lo > hi`) | `2 * dp[i+1][j-1] + 2` | 每个内部回文 P 能独立或被 `s[i]...s[j]` 包. **+2** 是新增 `"s[i]"` 和 `"s[i]s[j]"` 两种 |
    | **1 次** (`lo == hi`) | `2 * dp[i+1][j-1] + 1` | 同上但 `"s[i]"` 内部已有, 不重计. **+1** 是新增 `"s[i]s[j]"` |
    | **≥ 2 次** (`lo < hi`) | `2 * dp[i+1][j-1] - dp[lo+1][hi-1]` | 用外 `s[i]...s[j]` 包内部时, 跟用最内 `s[lo]...s[hi]` 包**重复**, 减掉 `dp[lo+1][hi-1]` |

    > **去重关键**: 当 s[i] 在内部多次出现时, 外层"s[i]...s[j]" 和内层最贴近的"s[lo]...s[hi]" 同字符可能产生**同一字符串**, 必须减.

5. **🔑 容斥后可能为负 → `(x % MOD + MOD) % MOD` / Handle negative after subtraction**

    模算术下减法可能让中间结果为负 (虽然真实数学值非负). C++ `%` 对负数返回负余数, 必须**加 MOD 再取模** 保正:

    ```cpp
    dp[i][j] = ((dp[i][j] % MOD) + MOD) % MOD;
    ```

    > 凡是 DP 涉及减法 + 模, 都要这个习惯写法. 漏掉 → WA.

6. **复杂度 O(n²) 状态 × O(n) 查 lo/hi = O(n³) / Complexity**

    Yang 的代码内层 `while (s[lo] != s[i])` 是 O(n). 可以**预处理**: 对每个 (i, c), 记录 c 在 i 右边/左边的最近位置 → 把 lo/hi 查询降到 O(1). Yang 的写法 O(n³) 在 LC 数据 (n ≤ 1000) 边缘但能过.

7. **按区间长度遍历 / Iterate by interval length**

    依赖 `dp[i+1][j]`, `dp[i][j-1]`, `dp[i+1][j-1]`, `dp[lo+1][hi-1]` 都是更短区间 → 按 `len` 由小到大填. 跟 [1000 合并石头](../1000-minimum-cost-to-merge-stones/README.md) / [1547 切棍](../1547-minimum-cost-to-cut-a-stick/README.md) 同套路.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int countPalindromicSubsequences(string s) {
            int n = s.size();
            const int MOD = 1e9 + 7;
            vector<vector<long long>> dp(n, vector<long long>(n, 0));
            for (int i = 0; i < n; i++) dp[i][i] = 1;                          // 单字符自身

            for (int len = 2; len <= n; len++) {                               // 按区间长度
                for (int i = 0; i + len - 1 < n; i++) {
                    int j = i + len - 1;
                    if (s[i] != s[j]) {
                        // 容斥
                        dp[i][j] = dp[i + 1][j] + dp[i][j - 1] - dp[i + 1][j - 1];
                    } else {
                        // 找内部最左/最右的 s[i]
                        int lo = i + 1, hi = j - 1;
                        while (lo <= hi && s[lo] != s[i]) lo++;
                        while (hi >= lo && s[hi] != s[i]) hi--;

                        if (lo > hi) {
                            // 内部无 s[i]: 内部回文可独立或被外包, 加 {"s[i]", "s[i]s[j]"}
                            dp[i][j] = 2 * dp[i + 1][j - 1] + 2;
                        } else if (lo == hi) {
                            // 内部恰一个 s[i]: "s[i]" 已被计, 只加 "s[i]s[j]"
                            dp[i][j] = 2 * dp[i + 1][j - 1] + 1;
                        } else {
                            // 内部 ≥ 2 个 s[i]: 减去重复部分 dp[lo+1][hi-1]
                            dp[i][j] = 2 * dp[i + 1][j - 1] - dp[lo + 1][hi - 1];
                        }
                    }
                    // 容斥可能负, 加 MOD 再取模
                    dp[i][j] = ((dp[i][j] % MOD) + MOD) % MOD;
                }
            }
            return (int)dp[0][n - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def countPalindromicSubsequences(self, s: str) -> int:
            n = len(s)
            MOD = 10**9 + 7
            # Python int 无大小限制, 但保持取模的好习惯
            dp = [[0] * n for _ in range(n)]
            for i in range(n):
                dp[i][i] = 1

            for length in range(2, n + 1):
                for i in range(n - length + 1):
                    j = i + length - 1
                    if s[i] != s[j]:
                        # 二维容斥
                        dp[i][j] = dp[i + 1][j] + dp[i][j - 1] - dp[i + 1][j - 1]
                    else:
                        # 找内部最左/最右 s[i]
                        lo, hi = i + 1, j - 1
                        while lo <= hi and s[lo] != s[i]:
                            lo += 1
                        while hi >= lo and s[hi] != s[i]:
                            hi -= 1

                        if lo > hi:
                            dp[i][j] = 2 * dp[i + 1][j - 1] + 2
                        elif lo == hi:
                            dp[i][j] = 2 * dp[i + 1][j - 1] + 1
                        else:
                            dp[i][j] = 2 * dp[i + 1][j - 1] - dp[lo + 1][hi - 1]
                    dp[i][j] %= MOD                                            # Python 取模对负数自然取正

            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var countPalindromicSubsequences = function(s) {
        const n = s.length;
        const MOD = 1_000_000_007n;                                            // BigInt 防中间溢出
        const dp = Array.from({length: n}, () => new Array(n).fill(0n));
        for (let i = 0; i < n; i++) dp[i][i] = 1n;

        for (let len = 2; len <= n; len++) {
            for (let i = 0; i + len - 1 < n; i++) {
                const j = i + len - 1;
                if (s[i] !== s[j]) {
                    dp[i][j] = dp[i + 1][j] + dp[i][j - 1] - dp[i + 1][j - 1];
                } else {
                    let lo = i + 1, hi = j - 1;
                    while (lo <= hi && s[lo] !== s[i]) lo++;
                    while (hi >= lo && s[hi] !== s[i]) hi--;
                    if (lo > hi)       dp[i][j] = 2n * dp[i + 1][j - 1] + 2n;
                    else if (lo === hi) dp[i][j] = 2n * dp[i + 1][j - 1] + 1n;
                    else               dp[i][j] = 2n * dp[i + 1][j - 1] - dp[lo + 1][hi - 1];
                }
                // BigInt 模, 加 MOD 防负
                dp[i][j] = ((dp[i][j] % MOD) + MOD) % MOD;
            }
        }
        return Number(dp[0][n - 1]);
    };
    ```

## Complexity

- **Time**: O(n³) — n² 状态 × O(n) 查 lo/hi. 可预处理"每字符的下一/上一位置" 表降到 O(n²).
- **Space**: O(n²).

## 相关题目

- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 回文**子串** (按位置) 计数, 简单很多
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 回文子序列**长度**, 不去重
- [0115. Distinct Subsequences](../0115-distinct-subsequences/README.md) — 双序列 distinct 计数, 同款"按值去重" 思维
- 0940\. Distinct Subsequences II (待补) — 同款"distinct" 计数 + 容斥, 不要求回文
- 0010\. Regular Expression Matching (待补) — 双序列 DP, 正则匹配
- 0044\. Wildcard Matching (待补) — 双序列 DP, 通配符匹配
