# 0115. Distinct Subsequences / 不同的子序列

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Double Sequence, Counting · 动态规划, 双序列, 计数
    - **Link**: [LeetCode](https://leetcode.com/problems/distinct-subsequences/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s` 和 `t`. 求 `t` 作为 `s` 的**子序列** 出现的**不同方案数** (位置不同算不同方案).

**中文**: 求 `t` 作为 `s` 子序列的方案数 (按字符位置区分).

## Key Insights

1. **🔑 双序列 + 计数 — 跟 [1143 LCS](../1143-longest-common-subsequence/README.md) 同模板, max → sum / Sum over choices instead of max**

    跟 LCS 同款双序列 `dp[i][j]` 结构, 但把 `max` 换成 `+=` — 这是 DP 从"最值" 到"计数" 的标志.

    | | [1143 LCS](../1143-longest-common-subsequence/README.md) | **0115 (本题)** |
    |---|---|---|
    | 问 | 最长公共子序列**长度** | t 作为 s 子序列的**方案数** |
    | 匹配时 | `dp[i][j] = dp[i-1][j-1] + 1` | **`dp[i][j] = dp[i-1][j-1] + dp[i-1][j]`** |
    | 不匹配时 | `dp[i][j] = max(...)` | **`dp[i][j] = dp[i-1][j]`** |

2. **状态: `dp[i][j] = "t 前 j 个字符" 在 "s 前 i 个字符" 中出现的方案数` / Prefix-to-prefix counting**

    "**t 必须完整出现**", 但 s 可以挑/跳任意字符. 求所有合法挑选方式的总数.

3. **🔑 匹配时两个来源相加 / Match: two ways contribute, sum them**

    `s[i-1] == t[j-1]` 时, 想以 `s[i-1]` 结尾的 s 前缀去匹配 `t[0..j)`, 有两种独立策略:

    - **用 `s[i-1]` 匹配 `t[j-1]`**: 那 t 的前 j-1 个就在 s 前 i-1 里找 → `dp[i-1][j-1]`
    - **不用 `s[i-1]`** (跳过 s 的这个字符): t 的前 j 个仍要在 s 前 i-1 里找 → `dp[i-1][j]`

    两策略**互不重叠** (一个用了 s[i-1], 一个没用) → 求和:

    $$dp[i][j] = dp[i-1][j-1] + dp[i-1][j]$$

    > **"用 / 不用" 互斥相加** 是计数 DP 的核心套路.

4. **不匹配时只能"跳" / Mismatch: only skip option remains**

    `s[i-1] != t[j-1]` 时, **不能** 用 s[i-1] 去匹配 t[j-1] → 只能跳过 s[i-1]:

    $$dp[i][j] = dp[i-1][j]$$

    > 跳的方向**只有一个** (跳 s, 不能跳 t) — 因为 t 必须完整出现, 一个字符都不能少.

5. **🔑 初始化: `dp[i][0] = 1` (空 t 在任何 s 前缀里都"出现 1 次") / Empty t base case**

    "空字符串是任何字符串的子序列", 而且**只有一种方式** ("挑 0 个"). 所以 `dp[i][0] = 1` 对所有 i 成立.

    `dp[0][j>0] = 0` (空 s 不可能产生非空 t, 默认 0 正好).

    > 漏写 `dp[i][0] = 1` 整个 dp 会全 0 — 计数 DP 的边界一定要走一遍.

6. **`unsigned long long` 防中间溢出 / Use unsigned long long**

    LC 题目保证最终答案 ≤ INT_MAX, 但**中间 `dp[i][j]` 可能 > INT_MAX**. 跟 [0377 Combination Sum IV](../0377-combination-sum-iv/README.md) 同一个坑. 用 `unsigned long long` (溢出 wrap) 或 `long long` (容量够) 都能 work; `int` 会 WA.

    > "答案在 int 内但中间会溢出" 是计数 DP 的高频翻车点, 见到大状态空间先用大容量整数试.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numDistinct(string s, string t) {
            int n = s.size(), m = t.size();
            vector<vector<unsigned long long>> dp(n + 1, vector<unsigned long long>(m + 1, 0));
            for (int i = 0; i <= n; i++) dp[i][0] = 1;             // 空 t 在任何 s 前缀里都"1 种"
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= m; j++) {
                    if (s[i - 1] == t[j - 1]) {
                        // 匹配: 用 s[i-1] (左上) + 不用 s[i-1] (上)
                        dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                    } else {
                        // 不匹配: 只能跳 s[i-1]
                        dp[i][j] = dp[i - 1][j];
                    }
                }
            }
            return (int)dp[n][m];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def numDistinct(self, s: str, t: str) -> int:
            # Python int 无大小限制, 不用考虑溢出
            n, m = len(s), len(t)
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            for i in range(n + 1):
                dp[i][0] = 1                                       # 空 t 总是 1 种方式
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if s[i - 1] == t[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]
                    else:
                        dp[i][j] = dp[i - 1][j]
            return dp[n][m]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {string} t
     * @return {number}
     */
    var numDistinct = function(s, t) {
        // JS Number 是 64-bit float, 整数精度到 2^53, LC 范围内安全
        const n = s.length, m = t.length;
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        for (let i = 0; i <= n; i++) dp[i][0] = 1;
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (s[i - 1] === t[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n][m];
    };
    ```

## Complexity

- **Time**: O(n × m).
- **Space**: O(n × m). (可压一维 — `j` 倒序遍历, 类似 [0/1 背包](../0416-partition-equal-subset-sum/README.md))

## 相关题目

- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — 同双序列模板, 求长度 (max) 而非计数 (sum)
- [0718. Maximum Length of Repeated Subarray](../0718-maximum-length-of-repeated-subarray/README.md) — 双序列子数组版
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 同款"计数 DP + 中间溢出" 陷阱
- 0392\. Is Subsequence (待补) — 判定 t 是不是 s 的子序列 (双指针即可)
- 0583\. Delete Operation for Two Strings (待补) — 最少删除次数, LCS 应用
- [0072. Edit Distance](../0072-edit-distance/README.md) — 双序列 DP 终极版, 三种操作
