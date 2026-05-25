# 0583. Delete Operation for Two Strings / 两个字符串的删除操作

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Double Sequence, LCS · 动态规划, 双序列, 最长公共子序列
    - **Link**: [LeetCode](https://leetcode.com/problems/delete-operation-for-two-strings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个字符串 `word1`, `word2`. 每次操作可以**删除任一字符串中的一个字符**, 求让两串相等的**最少操作数**.

**中文**: 每次删一个字符, 求让两串相等的最少操作数.

## Key Insights

1. **🔑 公式: 答案 = `n + m - 2 × LCS(word1, word2)` / Direct formula from LCS**

    最后两串相等的形态必然是它们的**最长公共子序列**. 要把 word1 变成 LCS, 删 `n - LCS` 个; word2 删 `m - LCS` 个. 加起来:

    $$\text{ans} = (n - L) + (m - L) = n + m - 2L,\quad L = \text{LCS}(word1, word2)$$

    > **见到"删字符让两串相等" 立刻反应 LCS 公式**. 这是 LCS 的标准应用题, 不用重新推 DP.

2. **代码就是 [1143 LCS](../1143-longest-common-subsequence/README.md) + 一行 / Same as 1143 plus one return**

    Yang 整段 DP 是 1143 一字不差; 只换 return:

    ```cpp
    return n + m - 2 * dp[n][m];
    ```

    > 看到能套模板就套, 不要自作主张换状态.

3. **为什么不能直接 `n + m - 2 × 公共字符数 (无序)` / Why LCS, not just shared chars**

    "ace" 和 "abcde" 共享 a, c, e 三个字符, 但是**顺序保持** (LCS 严格按原顺序匹配) — 不是"看字符集合" 而是"看子序列". 否则反例: word1="ab", word2="ba", 用集合大小 2 就错了 (实际 LCS=1, 答案 2).

4. **`swap` 优化是冗余 / The swap is cosmetic only**

    Yang 加了 `if (word1.size() < word2.size()) swap(...);`, 把长的放 word1. 在二维 dp 下**完全无影响** (转置而已, 答案对称). 仅当**一维滚动** 时才有意义 — 让"短的" 当内层 j 维度, 内存 `O(min(n,m))`. Yang 用二维, 这步可以删.

5. **替代解法: 直接 DP 求"最少删除" / Alternative: direct min-delete DP**

    不走 LCS 公式, 直接定义 `dp[i][j] = 让 word1[0..i) == word2[0..j) 的最少删除次数`:

    $$dp[i][j] = \begin{cases}
    dp[i-1][j-1], & word1[i-1] = word2[j-1] \\
    \min(dp[i-1][j],\ dp[i][j-1]) + 1, & \text{否则}
    \end{cases}$$

    边界: `dp[i][0] = i`, `dp[0][j] = j` (空一边 → 全删另一边). 答案 `dp[n][m]`.

    > 两种解法**完全等价**: 直接 DP 的 dp 值 = `i + j - 2 × LCS(word1[0..i), word2[0..j))`. LCS 公式更高级, 直接 DP 更朴素. 任选一种, 都会.

## Solution

=== "C++"
    === "v1 推荐: LCS + 公式"
        ```cpp
        class Solution {
        public:
            int minDistance(string word1, string word2) {
                int n = word1.size(), m = word2.size();
                vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
                for (int i = 1; i <= n; i++) {
                    for (int j = 1; j <= m; j++) {
                        if (word1[i - 1] == word2[j - 1]) {
                            dp[i][j] = dp[i - 1][j - 1] + 1;                   // 跟 1143 一字不差
                        } else {
                            dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                        }
                    }
                }
                return n + m - 2 * dp[n][m];                                   // 公式套
            }
        };
        ```

    === "v2: 直接 DP 求最少删除"
        ```cpp
        class Solution {
        public:
            int minDistance(string word1, string word2) {
                int n = word1.size(), m = word2.size();
                vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
                for (int i = 0; i <= n; i++) dp[i][0] = i;                     // 全删 word1 前 i 个
                for (int j = 0; j <= m; j++) dp[0][j] = j;                     // 全删 word2 前 j 个
                for (int i = 1; i <= n; i++) {
                    for (int j = 1; j <= m; j++) {
                        if (word1[i - 1] == word2[j - 1]) {
                            dp[i][j] = dp[i - 1][j - 1];                       // 末尾匹配, 不用删
                        } else {
                            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + 1;    // 删任一末尾, +1
                        }
                    }
                }
                return dp[n][m];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def minDistance(self, word1: str, word2: str) -> int:
            # v1 LCS + 公式
            n, m = len(word1), len(word2)
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if word1[i - 1] == word2[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1] + 1
                    else:
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
            return n + m - 2 * dp[n][m]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} word1
     * @param {string} word2
     * @return {number}
     */
    var minDistance = function(word1, word2) {
        const n = word1.length, m = word2.length;
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (word1[i - 1] === word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return n + m - 2 * dp[n][m];
    };
    ```

## Complexity

- **Time**: O(n × m).
- **Space**: O(n × m). (可压一维, 见 [1143 v2](../1143-longest-common-subsequence/README.md))

## 相关题目

- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — **本题就是它**, 加一行公式
- [1035. Uncrossed Lines](../1035-uncrossed-lines/README.md) — 同 LCS 应用 (变形)
- [0115. Distinct Subsequences](../0115-distinct-subsequences/README.md) — 双序列 DP 计数版
- [0072. Edit Distance](../0072-edit-distance/README.md) — **加 2 种操作** (插入 / 替换) 的终极版, 本题是它的特例 (只删)
- 0712\. Minimum ASCII Delete Sum for Two Strings (待补) — 本题加权版, 按 ASCII 计代价
- 0392\. Is Subsequence (待补) — 判定 t 是否 s 的子序列, LCS 退化版
