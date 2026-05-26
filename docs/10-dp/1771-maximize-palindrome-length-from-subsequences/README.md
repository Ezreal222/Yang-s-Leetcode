# 1771. Maximize Palindrome Length From Subsequences / 由子序列构造的最长回文串的长度

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval, String · 动态规划, 区间, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/maximize-palindrome-length-from-subsequences/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个字符串 `word1`, `word2`. 各选**一个非空子序列** `s1, s2`, 拼接 `s1 + s2` 形成回文. 求**最长可能长度**, 不可能则返 `0`.

**中文**: word1 + word2 各选非空子序列拼接成回文, 求最长长度. 不可能返 0.

## Key Insights

1. **🔑 转化: 拼接 + 跑 [0516 LPS](../0516-longest-palindromic-subsequence/README.md) + 端点约束 / Concat + LPS + endpoint constraint**

    设 `s = word1 + word2`. 在 `s` 上跑标准回文子序列 DP, 然后额外要求**回文的两端点必须分别在 word1 和 word2 里** — 这样 `s1` (在 word1 选) 和 `s2` (在 word2 选) 都非空.

    > **复合问题 = 已知模板 + 额外约束**, 是高级 DP 的常见结构. 这道题就是 0516 模板 + 端点跨界检查.

2. **状态: `dp[i][j] = s[i..j] 的最长回文子序列长度` (跟 [0516](../0516-longest-palindromic-subsequence/README.md) 一字不差) / Same as 0516**

    转移完全继承 0516:

    $$dp[i][j] = \begin{cases} dp[i+1][j-1] + 2, & s[i] = s[j] \\ \max(dp[i+1][j], dp[i][j-1]), & \text{otherwise} \end{cases}$$

3. **🔑 端点约束: 更新答案**只在**`s[i] == s[j]` 分支里 / Constraint only valid in match branch**

    Yang 的关键观察, 也是本题最容易翻车的地方:

    ```cpp
    if (s[i] == s[j]) {
        dp[i][j] = dp[i + 1][j - 1] + 2;
        if (i < word1.size() && j >= word1.size()) {            // 跨界检查
            ans = max(ans, dp[i][j]);                           // ⚠ 只能在这里更新
        }
    } else {
        dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);            // 不更新 ans
    }
    ```

    **为什么不能在 `else` 分支更新?** 因为 `dp[i][j]` 是"区间 `[i, j]` 内 LPS 长度", **不强制 LPS 端点就在 i 和 j**. 当 `s[i] != s[j]` 时, 实际最优 LPS 端点是某个 `(i', j')`, `i' > i` 或 `j' < j` — 这时 LPS 可能完全缩在 word1 或 word2 一边, **不满足跨界约束**.

    只有 `s[i] == s[j]` 且转移 `+2` 时, 我们才**确切知道** LPS 用了位置 i 和 j 作为端点. 此时检查 `i ∈ word1, j ∈ word2` 才有意义.

    > 这种"DP 状态值正确, 但答案抽取要看具体路径" 的题, 状态定义跟答案提取**解耦** 是常见陷阱.

4. **遍历顺序: i 倒序 j 顺序, 跟 0516 一致 / Same as 0516**

    依赖 `dp[i+1][j-1]`, `dp[i+1][j]`, `dp[i][j-1]` → i 必须先于 i+1 处理, 倒序; j 顺序.

5. **`ans = 0` 默认值兜底 / Default 0 if no cross-pair**

    若 word1 和 word2 没有任何公共字符 → 端点跨界的 `s[i] == s[j]` 永远不成立 → `ans` 始终 0. 默认初值 0 正好返回.

6. **空间 O((n+m)²) / Space**

    新 `s` 长度 `n + m`. dp 表 `(n+m)²`. LC 约束 word1.length + word2.length ≤ 1000 → 10^6 状态, 可接受.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int longestPalindrome(string word1, string word2) {
            string s = word1 + word2;
            int n = s.size();
            int w = word1.size();                                              // word1 边界
            vector<vector<int>> dp(n, vector<int>(n, 0));
            int ans = 0;

            for (int i = n - 1; i >= 0; i--) {                                 // i 倒序
                dp[i][i] = 1;
                for (int j = i + 1; j < n; j++) {                              // j 顺序
                    if (s[i] == s[j]) {
                        dp[i][j] = dp[i + 1][j - 1] + 2;
                        // ⚠ 端点更新只能在这里 — 此时 LPS 确实端点在 i 和 j
                        if (i < w && j >= w) {
                            ans = max(ans, dp[i][j]);
                        }
                    } else {
                        dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);            // 不更新 ans
                    }
                }
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def longestPalindrome(self, word1: str, word2: str) -> int:
            s = word1 + word2
            n = len(s)
            w = len(word1)
            dp = [[0] * n for _ in range(n)]
            ans = 0

            for i in range(n - 1, -1, -1):
                dp[i][i] = 1
                for j in range(i + 1, n):
                    if s[i] == s[j]:
                        dp[i][j] = dp[i + 1][j - 1] + 2
                        # 端点更新 — 只在 s[i] == s[j] 时, LPS 确实跨 (i, j)
                        if i < w <= j:                                         # Pythonic 链式比较
                            ans = max(ans, dp[i][j])
                    else:
                        dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])
            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} word1
     * @param {string} word2
     * @return {number}
     */
    var longestPalindrome = function(word1, word2) {
        const s = word1 + word2;
        const n = s.length, w = word1.length;
        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        let ans = 0;

        for (let i = n - 1; i >= 0; i--) {
            dp[i][i] = 1;
            for (let j = i + 1; j < n; j++) {
                if (s[i] === s[j]) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                    if (i < w && j >= w) {
                        ans = Math.max(ans, dp[i][j]);
                    }
                } else {
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O((n+m)²) where n, m = len(word1), len(word2).
- **Space**: O((n+m)²).

## 相关题目

- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 单串 LPS 母题, 本题直接套
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 回文子串计数
- [0730. Count Different Palindromic Subsequences](../0730-count-different-palindromic-subsequences/README.md) — 回文子序列 distinct 计数
- [1312. Minimum Insertion Steps to Make a String Palindrome](../1312-minimum-insertion-steps-to-make-a-string-palindrome/README.md) — 单串变回文最少插入
- 0005\. Longest Palindromic Substring (待补) — 最长回文子串
- 0132\. Palindrome Partitioning II (待补) — 最少分割使每段回文
