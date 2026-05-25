# 1035. Uncrossed Lines / 不相交的线

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Double Sequence, LCS · 动态规划, 双序列, 最长公共子序列
    - **Link**: [LeetCode](https://leetcode.com/problems/uncrossed-lines/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 两数组 `nums1`, `nums2` 上下排列. 把相等的数用一条直线连起来, 求最多能画多少条**不相交**的连线.

**中文**: 两数组上下排列, 把相等数用直线连起来 (相邻线不交叉), 求最多连线数.

## Key Insights

1. **🔑 题目变形, 本质就是 [1143 LCS](../1143-longest-common-subsequence/README.md) / This IS LCS in disguise**

    "不相交" + "保持原顺序" + "相等才能连" ⟺ 找一组**公共子序列**, 每对元素严格按原数组顺序对应. 一笔画**就是**一条 LCS 的元素配对.

    **代码跟 1143 一字不差** — Yang 直接套. 看到题先反应**"等价于哪个模板"**, 不要从零推导.

    > **解题第一步是分类**, 不是写代码. 这道题的全部价值就是练"识别变形" 这一步.

2. **变形的等价证明 / Why uncrossed ⟺ LCS**

    设答案是 k 条线, 把这 k 条线在 nums1 上的端点按下标递增排列 `i_1 < i_2 < ... < i_k`, nums2 对应端点也是 `j_1, j_2, ..., j_k`. 不相交条件 ⟺ `j_1 < j_2 < ... < j_k` (否则相邻两条会交叉).

    那么 `nums1[i_1..i_k] = nums2[j_1..j_k]` 且两边**下标都严格递增** → 这就是一条公共子序列. 反之, LCS 里每对配对自然下标递增, 连起来就是不相交.

    > 这是 reduction 的标准写法: 双向证明"该题的合法解 ⟺ LCS 的合法解".

3. **状态/转移/答案完全继承 1143 / Everything identical**

    见 [1143](../1143-longest-common-subsequence/README.md) Key Insights. 这里不重复.

    | 项 | 跟 1143 一样? |
    |---|---|
    | 状态 `dp[i][j]` | ✓ 前 i 个 + 前 j 个的 LCS |
    | 匹配转移 `+1` | ✓ |
    | 不匹配 `max(...)` | ✓ |
    | 答案 `dp[n][m]` | ✓ |
    | 滚动 O(min(n,m)) | ✓ 同 prevDiag 技巧 |

4. **唯一区别: 输入类型 / Only difference: input type**

    1143 是 `string`, 1035 是 `vector<int>`. 比较改 `text1[i-1] == text2[j-1]` 为 `nums1[i-1] == nums2[j-1]`. 仅此而已.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxUncrossedLines(vector<int>& nums1, vector<int>& nums2) {
            int n = nums1.size(), m = nums2.size();
            vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= m; j++) {
                    if (nums1[i - 1] == nums2[j - 1]) {
                        dp[i][j] = dp[i - 1][j - 1] + 1;                        // 匹配, 多一条线
                    } else {
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);             // 跳一个
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
        def maxUncrossedLines(self, nums1: list[int], nums2: list[int]) -> int:
            # 直接套 1143 LCS, 唯一改动: 字符串 → 数组比较
            n, m = len(nums1), len(nums2)
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if nums1[i - 1] == nums2[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1] + 1
                    else:
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
            return dp[n][m]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums1
     * @param {number[]} nums2
     * @return {number}
     */
    var maxUncrossedLines = function(nums1, nums2) {
        const n = nums1.length, m = nums2.length;
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (nums1[i - 1] === nums2[j - 1]) {
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
- **Space**: O(n × m).

## 相关题目

- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — **本题就是它**, 仅输入类型不同
- [0718. Maximum Length of Repeated Subarray](../0718-maximum-length-of-repeated-subarray/README.md) — 子数组版本 (强制连续)
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 单序列子序列
- 0583\. Delete Operation for Two Strings (待补) — LCS 的应用 (`n + m - 2 × LCS`)
- 0712\. Minimum ASCII Delete Sum for Two Strings (待补) — LCS 变体, 按 ASCII 权重
