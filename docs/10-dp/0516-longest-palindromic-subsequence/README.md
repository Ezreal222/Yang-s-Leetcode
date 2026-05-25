# 0516. Longest Palindromic Subsequence / 最长回文子序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval, String · 动态规划, 区间, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-palindromic-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s`, 返回**最长回文子序列** (Longest Palindromic Subsequence, LPS) 的长度. 子序列允许跳字符, 不要求连续.

**中文**: 求 `s` 的最长回文**子序列** 长度. 子序列允许跳.

## Key Insights

1. **🔑 跟 [0647 回文子串](../0647-palindromic-substrings/README.md) 的核心差别: 子序列 (允许跳) vs 子串 (必须连续) / Subseq vs Substring**

    跟 [0718 子数组](../0718-maximum-length-of-repeated-subarray/README.md) vs [1143 子序列 LCS](../1143-longest-common-subsequence/README.md) 同一对差别:

    | | [0647 子串](../0647-palindromic-substrings/README.md) | **0516 子序列 (本题)** |
    |---|---|---|
    | `dp[i][j]` 语义 | `s[i..j]` 是否回文 (bool) | `s[i..j]` 的 LPS 长度 (int) |
    | 两端相等时 | `s[i]==s[j] && dp[i+1][j-1]` (条件 AND) | `dp[i+1][j-1] + 2` (内部最优 +2) |
    | **两端不等时** | **`false`** (段断了) | **`max(dp[i+1][j], dp[i][j-1])`** (跳一端) |
    | 答案 | `count(dp[*][*] == true)` | `dp[0][n-1]` (整段最优) |

    > 子串 (连续) → 不匹配段直接断; 子序列 (允许跳) → 不匹配跳一端继续. **这是序列 DP 的统一规律**.

2. **状态: `dp[i][j] = s[i..j] 中 LPS 的长度` / Length of LPS in interval [i, j]**

    跟 0647 一样, `i, j` 是**区间端点**, 不是结尾. 区间 DP.

3. **🔑 转移: 两种情况 / Two cases**

    用[最后一步](../topic-dp-thinking-process.md) 思维: `s[i], s[j]` 这两个端点字符在 LPS 里**用不用**?

    - **`s[i] == s[j]`**: 可以两端都用 → `LPS(s[i..j]) = LPS(s[i+1..j-1]) + 2`
        - 内部最长回文加上首尾两字符 (互相对称)
    - **`s[i] != s[j]`**: 至少一个不能出现在 LPS 末位 → 跳一个继续:
        - 跳左 `s[i]`: `dp[i+1][j]`
        - 跳右 `s[j]`: `dp[i][j-1]`
        - 取 max

    $$dp[i][j] = \begin{cases} dp[i+1][j-1] + 2, & s[i] = s[j] \\ \max(dp[i+1][j], dp[i][j-1]), & \text{否则} \end{cases}$$

4. **🔑 边界 `dp[i][i] = 1` 写在 i 循环里 / Diagonal initialization inside i loop**

    Yang 把 `dp[i][i] = 1` 放在外层 i 循环开头 — 单字符自身就是长度 1 的回文. 这是**单字符回文** 的边界, 必须显式写, 否则 `dp[i][i]` 默认 0 → 整个推导从 0 起步, 错.

    `j < i` 的格不会用到 (j ≥ i+1 才进内层), 不用管.

    > 等价写法是单独一个 `for i: dp[i][i] = 1` 在主循环之前. Yang 嵌进去更紧凑.

5. **🔑 遍历顺序: 跟 [0647](../0647-palindromic-substrings/README.md) 完全一致 / Same traversal direction**

    `dp[i][j]` 依赖三个邻居:

    - `dp[i+1][j-1]` (左下): i 更大, j 更小
    - `dp[i+1][j]` (下): i 更大, j 同
    - `dp[i][j-1]` (左): i 同, j 更小

    都要先于 `dp[i][j]` 算好 → **外层 i 倒序, 内层 j 顺序 (从 i+1 开始)**.

    > 同款区间 DP. **i 倒, j 顺** 是这类题的口诀.

6. **答案在 `dp[0][n-1]` / Answer at the full-interval corner**

    跟 0647 (扫全表数 true) 不同, 这里就是"整个串" 的 LPS, 直接读角落.

7. **替代视角: 0516 ≡ LCS(s, reverse(s)) / Reduction to LCS**

    `s` 的最长回文子序列长度 = `s` 和 `reverse(s)` 的最长公共子序列长度 (LCS). 直接调用 [1143](../1143-longest-common-subsequence/README.md) 的 LCS 代码也能解, 思路是"回文 = 正读 = 反读", 公共部分就是回文.

    但直接区间 DP 更高效 (省了 reverse), 也更直接. 留作备选思路.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int longestPalindromeSubseq(string s) {
            int n = s.size();
            vector<vector<int>> dp(n, vector<int>(n, 0));
            for (int i = n - 1; i >= 0; i--) {                     // ⚠ i 倒序
                dp[i][i] = 1;                                      // 单字符回文长度 1
                for (int j = i + 1; j < n; j++) {                  // j 从 i+1 开始
                    if (s[i] == s[j]) {
                        dp[i][j] = dp[i + 1][j - 1] + 2;           // 两端都用
                    } else {
                        dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]); // 跳一端
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
        def longestPalindromeSubseq(self, s: str) -> int:
            n = len(s)
            dp = [[0] * n for _ in range(n)]
            for i in range(n - 1, -1, -1):                         # i 倒序
                dp[i][i] = 1
                for j in range(i + 1, n):
                    if s[i] == s[j]:
                        dp[i][j] = dp[i + 1][j - 1] + 2
                    else:
                        dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])
            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var longestPalindromeSubseq = function(s) {
        const n = s.length;
        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        for (let i = n - 1; i >= 0; i--) {
            dp[i][i] = 1;
            for (let j = i + 1; j < n; j++) {
                if (s[i] === s[j]) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[0][n - 1];
    };
    ```

## Complexity

- **Time**: O(n²).
- **Space**: O(n²).

## 相关题目

- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — **子串版**, 同区间 DP 但不匹配段直接断
- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — `LPS(s) ≡ LCS(s, reverse(s))`, 等价解法
- 0005\. Longest Palindromic Substring (待补) — 求最长回文**子串**, 跟 0647 同套路
- 0132\. Palindrome Partitioning II (待补) — 最少分割使每段回文, 区间 DP
- 0730\. Count Different Palindromic Subsequences (待补) — 进阶: 数不同回文子序列个数
- 0312\. Burst Balloons (待补) — 经典区间 DP, 进阶应用
