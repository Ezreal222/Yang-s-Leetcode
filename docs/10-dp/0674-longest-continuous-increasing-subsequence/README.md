# 0674. Longest Continuous Increasing Subsequence / 最长连续递增序列

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, Array · 动态规划, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-continuous-increasing-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 求**最长连续递增子数组** 的长度 (必须严格递增, 元素**位置连续**).

**中文**: 最长连续严格递增**子数组** 的长度 (位置必须挨着).

## Key Insights

1. **🔑 跟 [0300 LIS](../0300-longest-increasing-subsequence/README.md) 的核心差别: 子数组 vs 子序列 / Subarray vs Subsequence**

    | 题 | 类型 | dp[i] 看谁 | 复杂度 |
    |---|---|---|---|
    | [0300](../0300-longest-increasing-subsequence/README.md) | **子序列** (允许间隔) | 所有 `j < i` 且 `nums[j] < nums[i]` | O(n²) DP / O(n log n) patience |
    | **0674 (本题)** | **子数组** (位置连续) | **只看 `dp[i-1]`** | O(n) |

    "**连续**" 这一字让转移从 O(i) 暴跌到 O(1) — 因为新元素只能接在上一个元素后, 中间不能跳.

2. **状态: `dp[i] = 以 nums[i] 结尾的最长 LCIS` (跟 0300 同款"以 i 结尾") / Same "ending at i" template**

    继承 [0300](../0300-longest-increasing-subsequence/README.md) 的"以 i 结尾" 状态模式. 答案仍是 `max(dp[*])`, 不是 `dp[n-1]`.

3. **转移: 一行 / Transition**

    $$dp[i] = \begin{cases} dp[i-1] + 1, & nums[i] > nums[i-1] \\ 1, & \text{否则} \end{cases}$$

    Yang 用 `dp[i] = max(dp[i], dp[i-1] + 1)`, 但因为 `dp[i]` 初值 1, `dp[i-1] + 1 ≥ 2 > 1`, 这个 max 其实多余 — 直接赋值就行. 写 max 也没错, 只是冗余.

4. **可压到 O(1) 空间 / Rolling single var (gross)**

    `dp[i]` 只依赖 `dp[i-1]` → 一个变量 `cur` 足够:

    ```cpp
    int cur = 1, result = 1;
    for (int i = 1; i < n; i++) {
        cur = (nums[i] > nums[i - 1]) ? cur + 1 : 1;
        result = max(result, cur);
    }
    ```

5. **贪心也行 (本质同 DP) / Greedy = same algorithm**

    可以不写 dp 数组, 直接维护当前递增段长度. 数学上跟 DP 等价 — 状态压成单变量后, DP 退化成贪心.

    > 这是 DP / 贪心边界的小案例: **当 dp 状态可压到 O(1) 且转移由"当前比较" 决定, DP 跟贪心几乎是同一份代码**.

## Solution

=== "C++"
    === "v1 (Yang 原版): dp 数组"
        ```cpp
        class Solution {
        public:
            int findLengthOfLCIS(vector<int>& nums) {
                int n = nums.size();
                vector<int> dp(n, 1);
                int result = 1;
                for (int i = 1; i < n; i++) {
                    if (nums[i] > nums[i - 1]) dp[i] = dp[i - 1] + 1;    // max 多余, 直接赋
                    result = max(result, dp[i]);
                }
                return result;
            }
        };
        ```

    === "v2: 滚动 O(1) (贪心写法)"
        ```cpp
        class Solution {
        public:
            int findLengthOfLCIS(vector<int>& nums) {
                int cur = 1, result = 1;
                for (int i = 1; i < (int)nums.size(); i++) {
                    cur = (nums[i] > nums[i - 1]) ? cur + 1 : 1;          // 接上 / 断了重新计
                    result = max(result, cur);
                }
                return result;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def findLengthOfLCIS(self, nums: list[int]) -> int:
            # 滚动 O(1) 版, Pythonic 一遍扫
            cur = result = 1
            for i in range(1, len(nums)):
                cur = cur + 1 if nums[i] > nums[i - 1] else 1
                result = max(result, cur)
            return result
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var findLengthOfLCIS = function(nums) {
        let cur = 1, result = 1;
        for (let i = 1; i < nums.length; i++) {
            cur = nums[i] > nums[i - 1] ? cur + 1 : 1;
            result = Math.max(result, cur);
        }
        return result;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 子序列版 (允许间隔), 难度大幅升级
- 0673\. Number of Longest Increasing Subsequence (待补) — LIS 个数
- 0053\. Maximum Subarray — 同款"子数组 + 以 i 结尾" 模板, 求最大和 → [§09 0053](../../09-greedy/0053-maximum-subarray/README.md)
- 0978\. Longest Turbulent Subarray (待补) — 子数组 + 交替增减
- 1567\. Maximum Length of Subarray With Positive Product (待补) — 同款连续子数组 + 状态分类
