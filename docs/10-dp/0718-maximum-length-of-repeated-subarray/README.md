# 0718. Maximum Length of Repeated Subarray / 最长重复子数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Double Sequence · 动态规划, 双序列
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-length-of-repeated-subarray/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个数组 `nums1` 和 `nums2`, 求两者**共同出现** 的最长**子数组** (连续) 长度.

**中文**: 两数组共同出现的最长**连续**子数组长度.

## Key Insights

1. **🔑 入门双序列 DP / Seed for two-sequence DP**

    跟一维序列 DP ([0300](../0300-longest-increasing-subsequence/README.md) / [0674](../0674-longest-continuous-increasing-subsequence/README.md)) 不同, 双序列 DP 用**二维 dp**, 横纵分别代表两个序列的指针位置.

    > 看到"两个字符串/数组" + "最长/最短/匹配数" → 想 `dp[i][j]` 二维.

2. **状态: `dp[i][j] = 以 nums1[i-1], nums2[j-1] 同时结尾的最长公共子数组` (强制双端结尾) / dp[i][j] ending at both i-1 and j-1**

    跟 [0300 LIS](../0300-longest-increasing-subsequence/README.md) 的"以 i 结尾" 同思路, 升级到双序列 → "**以 nums1[i-1] 和 nums2[j-1] 同时结尾**". 强制两个指针都用上当前元素, 这样:

    - **连续约束** 自动成立 (前一对必须是 `i-1, j-1` 同时往前一步).
    - **不匹配时 `dp[i][j] = 0`** (不能跳过当前元素, 段直接断).

    > **强制结尾** 是子数组 / 公共子数组 DP 的标志. 后面 [1143 LCS](../1143-longest-common-subsequence/README.md) 求子序列就不强制结尾, 转移完全不同.

3. **🔑 哨兵技巧: dp 开 `(n+1) × (m+1)`, 索引 i, j 从 1 开始 / Sentinel row+col simplifies boundary**

    Yang 用了 `dp(n + 1, vector<int>(m + 1, 0))`, 索引 `i ∈ [1, n], j ∈ [1, m]`, **`nums1[i-1]` 对应 `dp[i][j]`** (注意 -1 偏移).

    好处: `dp[0][*]` 和 `dp[*][0]` 都自然是 0 (其中一个数组空时, 公共子数组长度 0), **不需要单独处理 `i=0` 或 `j=0` 的边界**.

    > 这是处理"前 i 个 / 前 j 个" 类双序列 DP 的标配技巧. 看到 `nums[i-1]` 就知道在用哨兵.

4. **转移: 匹配时 +1, 不匹配时 0 / Match or break**

    $$dp[i][j] = \begin{cases} dp[i-1][j-1] + 1, & nums1[i-1] = nums2[j-1] \\ 0, & \text{否则} \end{cases}$$

    Yang 的代码省了 else 分支 (因为 dp 初值已经是 0), 直接 `if (match) dp[i][j] = dp[i-1][j-1] + 1`.

5. **答案是 `max(dp[*][*])` 不是 `dp[n][m]` / Answer scans full grid**

    跟[0300](../0300-longest-increasing-subsequence/README.md) 的"以 i 结尾" 同理: 子数组可以在任何位置结束 → 必须扫整张 dp 表取最大. 不要返回 `dp[n][m]` (那只对应"以 nums1 最后一位 + nums2 最后一位结尾").

6. **可压到 O(min(n, m)) 一维 / Rolling 1D with backward j**

    `dp[i][j]` 只依赖 `dp[i-1][j-1]` (左上对角). 用一维数组 + **`j` 倒序** (类似 [0/1 背包](../0416-partition-equal-subset-sum/README.md)) 防止覆盖:

    ```cpp
    vector<int> dp(m + 1, 0);
    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= 1; j--) {                     // 倒序: 保护 dp[j-1] (即 dp[i-1][j-1])
            if (nums1[i-1] == nums2[j-1]) dp[j] = dp[j-1] + 1;
            else                          dp[j] = 0;       // 这里 else 不能省, 一维下要主动清零
        }
    }
    ```

    > 一维下 **else 不能省** — 因为 `dp[j]` 是上一行的值 (脏), 不主动清零会保留错的旧值.

## Solution

=== "C++"
    === "v1 推荐: 二维 DP (易读)"
        ```cpp
        class Solution {
        public:
            int findLength(vector<int>& nums1, vector<int>& nums2) {
                int n = nums1.size(), m = nums2.size();
                vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));  // 哨兵
                int result = 0;
                for (int i = 1; i <= n; i++) {
                    for (int j = 1; j <= m; j++) {
                        // 强制以 nums1[i-1], nums2[j-1] 同时结尾
                        if (nums1[i - 1] == nums2[j - 1]) {
                            dp[i][j] = dp[i - 1][j - 1] + 1;
                        }
                        // else dp[i][j] = 0 (默认值已是 0, 省略)
                        result = max(result, dp[i][j]);
                    }
                }
                return result;
            }
        };
        ```

    === "v2: 滚动一维 O(min(n,m))"
        ```cpp
        class Solution {
        public:
            int findLength(vector<int>& nums1, vector<int>& nums2) {
                int n = nums1.size(), m = nums2.size();
                vector<int> dp(m + 1, 0);
                int result = 0;
                for (int i = 1; i <= n; i++) {
                    for (int j = m; j >= 1; j--) {                     // ⚠ 倒序保护左上角
                        if (nums1[i - 1] == nums2[j - 1]) dp[j] = dp[j - 1] + 1;
                        else                              dp[j] = 0;    // ⚠ 一维下不能省
                        result = max(result, dp[j]);
                    }
                }
                return result;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def findLength(self, nums1: list[int], nums2: list[int]) -> int:
            n, m = len(nums1), len(nums2)
            # 哨兵行/列: 前后多一格全 0, 简化边界
            # 等价 C++ vector<vector<int>>(n+1, vector<int>(m+1, 0))
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            result = 0
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if nums1[i - 1] == nums2[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1] + 1
                        if dp[i][j] > result:
                            result = dp[i][j]
            return result
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums1
     * @param {number[]} nums2
     * @return {number}
     */
    var findLength = function(nums1, nums2) {
        const n = nums1.length, m = nums2.length;
        // Array.from 创建独立每行, 防 fill 共享引用
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        let result = 0;
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (nums1[i - 1] === nums2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                    if (dp[i][j] > result) result = dp[i][j];
                }
            }
        }
        return result;
    };
    ```

## Complexity

- **Time**: O(n × m).
- **Space**: O(n × m) (v1) / O(min(n, m)) (v2).

## 相关题目

- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 单序列"以 i 结尾", 本题的"双序列升级版"
- [0674. Longest Continuous Increasing Subsequence](../0674-longest-continuous-increasing-subsequence/README.md) — 同款"子数组 + 强制结尾"
- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — **子序列版**, 双序列 DP 的兄弟; 不匹配时 `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`
- 0583\. Delete Operation for Two Strings (待补) — LCS 的应用
- 0072\. Edit Distance (待补) — 双序列 DP 终极版, 三种操作的最少代价
- 0005\. Longest Palindromic Substring (待补) — 单序列"子数组" 但要回文
