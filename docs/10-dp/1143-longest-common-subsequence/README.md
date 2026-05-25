# 1143. Longest Common Subsequence / 最长公共子序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Double Sequence, LCS · 动态规划, 双序列, 最长公共子序列
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-common-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个字符串 `text1`, `text2`. 求**最长公共子序列** (Longest Common Subsequence, LCS) 的长度. 子序列允许跳过任意字符, 但保持原顺序.

**中文**: 两字符串最长公共子序列长度. 子序列允许跳过, 不要求连续.

## Key Insights

1. **🔑 跟 [0718](../0718-maximum-length-of-repeated-subarray/README.md) 的核心差别: 子序列 (允许跳) vs 子数组 (必须连续) / Subsequence vs Subarray**

    一字之差, 状态/转移/答案三处都改:

    | | [0718 (子数组)](../0718-maximum-length-of-repeated-subarray/README.md) | **1143 (子序列, 本题)** |
    |---|---|---|
    | 状态语义 | `dp[i][j]` = 以 `text1[i-1], text2[j-1]` **同时结尾** | `dp[i][j]` = `text1[0..i)` 与 `text2[0..j)` 的 LCS (不强制结尾) |
    | 匹配时 | `dp[i-1][j-1] + 1` | `dp[i-1][j-1] + 1` (一样) |
    | **不匹配时** | **`= 0`** (段断了) | **`= max(dp[i-1][j], dp[i][j-1])`** (跳一个继续) |
    | 答案 | `max(dp[*][*])` 全表扫 | **`dp[n][m]`** 角落 |

    > 看到"子序列" 就要立刻反应"不匹配能 fall back 到 `dp[i-1][j]` 或 `dp[i][j-1]`". 这是 LCS 模板的灵魂.

2. **状态: `dp[i][j] = text1 前 i 个字符 + text2 前 j 个字符的 LCS 长度` (不强制结尾) / Prefix-based, not end-forced**

    跟 0718 不同, 这里**不要求** `text1[i-1]`, `text2[j-1]` 必须出现在 LCS 里 — 算的是"考虑了前 i, 前 j 个字符" 的最优.

    这种"前 i + 前 j" 的状态定义让答案直接等于 `dp[n][m]` (考虑了全部字符), 而非全表扫最大.

3. **🔑 转移用"最后一步" 思维 / Last-step thinking**

    LCS 的最后一对字符 `text1[i-1], text2[j-1]` 状态有 2 种:

    - **相等**: 它俩可以是 LCS 末尾 → `dp[i][j] = dp[i-1][j-1] + 1`.
    - **不等**: 至少一个不在 LCS 末尾 → 跳掉一个继续:
        - 跳 `text1[i-1]`: 看 `dp[i-1][j]`
        - 跳 `text2[j-1]`: 看 `dp[i][j-1]`
        - 取 max.

    > 这就是为什么"子序列" 能 fall back — 因为跳掉一个不影响"找前缀里的 LCS".

4. **不匹配时为什么不取 `dp[i-1][j-1]`? / Why not also include dp[i-1][j-1] in else?**

    `dp[i-1][j-1] ≤ dp[i-1][j]` (后者多考虑了一个字符, 至少不会更差), 同理 `dp[i-1][j-1] ≤ dp[i][j-1]`. 所以**自动被两个相邻值覆盖**, 不用单独写. 多写也没错, 但冗余.

5. **哨兵 `(n+1) × (m+1)` 同 0718 / Same sentinel trick**

    `dp[0][*]` 和 `dp[*][0]` 是 0 (一边空时 LCS 0). 索引 i, j 从 1 开始, `text1[i-1]` 是当前比较字符. 跟 [0718](../0718-maximum-length-of-repeated-subarray/README.md) 一致.

6. **可压到 O(min(n, m)) 一维 / Roll to 1D (tricky, needs prev_diag)**

    `dp[i][j]` 依赖三个邻居: `dp[i-1][j-1]` (左上), `dp[i-1][j]` (上), `dp[i][j-1]` (左). 一维压缩需要一个额外变量 `prevDiag` 保存"左上角" — 因为更新 `dp[j]` 时, 旧值就是"上", 而"左上" 已经被上一轮覆盖了.

    ```cpp
    vector<int> dp(m + 1, 0);
    for (int i = 1; i <= n; i++) {
        int prevDiag = 0;                      // dp[i-1][0]
        for (int j = 1; j <= m; j++) {
            int tmp = dp[j];                   // 保存当前 dp[j] (即 dp[i-1][j], 即"上")
            if (text1[i-1] == text2[j-1]) dp[j] = prevDiag + 1;
            else                          dp[j] = max(dp[j], dp[j-1]);
            prevDiag = tmp;                    // 给下一轮用
        }
    }
    return dp[m];
    ```

    > 比 [0/1 背包](../0416-partition-equal-subset-sum/README.md) 的一维压缩更绕 — 多了"左上" 这一个依赖. 数据规模允许就用二维, 别硬压.

## Solution

=== "C++"
    === "v1 推荐: 二维 DP (易读)"
        ```cpp
        class Solution {
        public:
            int longestCommonSubsequence(string text1, string text2) {
                int n = text1.size(), m = text2.size();
                vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
                for (int i = 1; i <= n; i++) {
                    for (int j = 1; j <= m; j++) {
                        if (text1[i - 1] == text2[j - 1]) {
                            dp[i][j] = dp[i - 1][j - 1] + 1;                   // 末尾匹配, +1
                        } else {
                            dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);        // 跳一个
                        }
                    }
                }
                return dp[n][m];                                                // 角落即答案
            }
        };
        ```

    === "v2: 一维 O(min(n,m)) (需 prevDiag 暂存)"
        ```cpp
        class Solution {
        public:
            int longestCommonSubsequence(string text1, string text2) {
                int n = text1.size(), m = text2.size();
                vector<int> dp(m + 1, 0);
                for (int i = 1; i <= n; i++) {
                    int prevDiag = 0;                                          // dp[i-1][0]
                    for (int j = 1; j <= m; j++) {
                        int tmp = dp[j];                                       // 旧 dp[j] = 上一行同列
                        if (text1[i - 1] == text2[j - 1]) dp[j] = prevDiag + 1;
                        else                              dp[j] = max(dp[j], dp[j - 1]);
                        prevDiag = tmp;                                        // 给下一轮当左上
                    }
                }
                return dp[m];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def longestCommonSubsequence(self, text1: str, text2: str) -> int:
            n, m = len(text1), len(text2)
            # 哨兵 (n+1) × (m+1) 全 0, 简化边界
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if text1[i - 1] == text2[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1] + 1
                    else:
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
            return dp[n][m]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} text1
     * @param {string} text2
     * @return {number}
     */
    var longestCommonSubsequence = function(text1, text2) {
        const n = text1.length, m = text2.length;
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (text1[i - 1] === text2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[n][m];
    };
    ```

## Complexity

- **Time**: O(n × m).
- **Space**: O(n × m) (v1) / O(min(n, m)) (v2).

## 相关题目

- [0718. Maximum Length of Repeated Subarray](../0718-maximum-length-of-repeated-subarray/README.md) — 子数组版, 强制双端结尾
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 单序列"子序列" 母题
- 0583\. Delete Operation for Two Strings (待补) — `min ops = n + m - 2 × LCS`, 直接套本题
- 0072\. Edit Distance (待补) — 双序列 DP 终极版, 三种操作
- 0392\. Is Subsequence (待补) — 判定版, `dp` 可省, 双指针即可
- [1035. Uncrossed Lines](../1035-uncrossed-lines/README.md) — 题面变形, 本质完全等价 LCS
- 0712\. Minimum ASCII Delete Sum for Two Strings (待补) — LCS 变体, 按 ASCII 权重
