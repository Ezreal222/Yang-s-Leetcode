# 1312. Minimum Insertion Steps to Make a String Palindrome / 让字符串成为回文串的最少插入次数

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval, String · 动态规划, 区间, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s`. 每次可以在**任意位置插入任意字符**. 求把 `s` 变成回文的**最少插入次数**.

**中文**: 任意位置插入字符, 求让 `s` 变回文的最少插入次数.

## Key Insights

1. **🔑 跟 [0516 LPS](../0516-longest-palindromic-subsequence/README.md) 是镜像题 / Mirror of 0516**

    转移结构跟 0516 几乎一样, 只是"求最大长度" 换成"求最少插入". 用[最后一步](../topic-dp-thinking-process.md) 思维考虑 `s[i], s[j]`:

    | | [0516 LPS (max)](../0516-longest-palindromic-subsequence/README.md) | **1312 (min)** |
    |---|---|---|
    | 两端相等 | `dp[i][j] = dp[i+1][j-1] + 2` | `dp[i][j] = dp[i+1][j-1]` |
    | 两端不等 | `max(dp[i+1][j], dp[i][j-1])` | `min(dp[i+1][j], dp[i][j-1]) + 1` |

    > **同结构区间 DP, max → min, +2 → 不动, max → min+1**. 双胞胎.

2. **状态: `dp[i][j] = 把 s[i..j] 变成回文的最少插入数` / Min insertions for s[i..j]**

3. **🔑 转移: 看两端 / Look at endpoints**

    - **`s[i] == s[j]`**: 两端已匹配, **不用插**; 处理内部 → `dp[i][j] = dp[i+1][j-1]`.
    - **`s[i] != s[j]`**: 必须插一次让某一端匹配, 取较小:
        - 在**右边插 s[i]** 让它跟左端 `s[i]` 配对 → 剩内部 `s[i+1..j]` 要处理 → `dp[i+1][j] + 1`
        - 在**左边插 s[j]** 让它跟右端 `s[j]` 配对 → 剩内部 `s[i..j-1]` 要处理 → `dp[i][j-1] + 1`
        - `dp[i][j] = min(...) + 1`

4. **边界 `dp[i][i] = 0` (单字符已是回文) — 默认值正好 / Single char needs zero**

    Yang 没显式写, 因为 vector 默认初值 0, 而且 `j > i` 才进内层循环 → 单字符状态默认 0, 不用动. 比 0516 还省一步 (那里要 `dp[i][i] = 1`).

5. **遍历顺序: 跟 [0516](../0516-longest-palindromic-subsequence/README.md)/[0647](../0647-palindromic-substrings/README.md) 同 — i 倒序 j 顺序 / Same reverse-i forward-j**

    `dp[i][j]` 依赖 `dp[i+1][...]` (下) 和 `dp[...][j-1]` (左) — 都是"更大 i / 更小 j" 方向. **i 必须先于 i+1 之后处理**, 倒序; j 顺序.

6. **🔑 等价视角: `ans = n - LPS(s)` / Alternative: subtract LPS length**

    每个字符要么在最终回文里被"配对" (属于 LPS), 要么"被新插入字符配对". `n - LPS` 个字符在原串里没配上 → 每个需要 1 次插入.

    所以可以**直接调用 [0516](../0516-longest-palindromic-subsequence/README.md) 求 LPS, 再 `n - LPS`** — 跟 [0583 Delete](../0583-delete-operation-for-two-strings/README.md) 用 `n + m - 2 × LCS` 同思路.

    两种解法**完全等价**, 代码量也差不多. 选哪种都行 — Yang 用直接 DP, 思路更直接.

## Solution

=== "C++"
    === "v1 推荐: 直接区间 DP (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int minInsertions(string s) {
                int n = s.size();
                vector<vector<int>> dp(n, vector<int>(n, 0));      // 单字符默认 0
                for (int i = n - 1; i >= 0; i--) {                 // i 倒序
                    for (int j = i + 1; j < n; j++) {              // j 从 i+1 开始
                        if (s[i] == s[j]) {
                            dp[i][j] = dp[i + 1][j - 1];           // 两端已配
                        } else {
                            dp[i][j] = min(dp[i + 1][j], dp[i][j - 1]) + 1;
                        }
                    }
                }
                return dp[0][n - 1];
            }
        };
        ```

    === "v2: n - LPS 视角 (调 0516)"
        ```cpp
        class Solution {
        public:
            int minInsertions(string s) {
                int n = s.size();
                vector<vector<int>> dp(n, vector<int>(n, 0));
                for (int i = n - 1; i >= 0; i--) {
                    dp[i][i] = 1;
                    for (int j = i + 1; j < n; j++) {
                        if (s[i] == s[j]) dp[i][j] = dp[i + 1][j - 1] + 2;
                        else              dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
                    }
                }
                return n - dp[0][n - 1];                           // 答案 = n - LPS
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def minInsertions(self, s: str) -> int:
            # v1 直接 DP, 跟 0516 结构一样, max → min, +2 → 不动, max → min+1
            n = len(s)
            dp = [[0] * n for _ in range(n)]
            for i in range(n - 1, -1, -1):                         # i 倒序
                for j in range(i + 1, n):
                    if s[i] == s[j]:
                        dp[i][j] = dp[i + 1][j - 1]
                    else:
                        dp[i][j] = min(dp[i + 1][j], dp[i][j - 1]) + 1
            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var minInsertions = function(s) {
        const n = s.length;
        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        for (let i = n - 1; i >= 0; i--) {
            for (let j = i + 1; j < n; j++) {
                if (s[i] === s[j]) {
                    dp[i][j] = dp[i + 1][j - 1];
                } else {
                    dp[i][j] = Math.min(dp[i + 1][j], dp[i][j - 1]) + 1;
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

- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — **镜像题** (max → min), `ans = n - LPS`
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 入门
- [0583. Delete Operation for Two Strings](../0583-delete-operation-for-two-strings/README.md) — 同款"LCS 公式" 思路 (`n + m - 2 × LCS`)
- 0132\. Palindrome Partitioning II (待补) — 最少分割使每段回文
- 0005\. Longest Palindromic Substring (待补) — 求最长回文**子串**, 同区间 DP 或扩散
- 0680\. Valid Palindrome II (待补) — 允许删一个字符的判定 (双指针)
