# 0416. Partition Equal Subset Sum / 分割等和子集

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, 0/1 Knapsack · 动态规划, 01 背包
    - **Link**: [LeetCode](https://leetcode.com/problems/partition-equal-subset-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给只含正整数的数组 `nums`, 判断能否分成两个子集, 使两子集**和相等**.

**中文**: 正整数数组 `nums`, 能否分成两个子集和相等.

## Key Insights

1. **问题转化: 找一个子集 = total/2 / Reduce to subset-sum-to-target**

    两堆和相等 ⟺ 每堆 `= total / 2`. 只要找出**一个**和恰为 `total/2` 的子集, 剩下自动凑齐另一半. 没找到则 false.

    > 凡是"分两组相等" 的题先这么转 — 把"找两堆" 简化成"凑一个目标值".

2. **奇数早返 / Odd total ⇒ impossible**

    `total` 是奇数, `total/2` 不是整数, 直接 false. 省后续 O(n·target) 计算.

3. **这就是 0/1 背包模板 / This is the canonical 0/1 knapsack**

    把每个 `num` 看成一件物品: **重量 = 价值 = num**, 容量 = `target = total/2`. 问"能否把容量恰好装满":

    - 装满 ⟺ `dp[target] == target` (最大价值能等于容量)
    - 装不满 ⟺ `dp[target] < target`

    > 一旦看出"选/不选" + "容量约束" + "每个元素最多用一次" → 立即套 0/1 背包.

4. **状态: dp[j] = 容量 j 时能装的最大价值 (= 最大子集和 ≤ j) / dp[j] = max sum reachable not exceeding j**

    转移: 物品 `n` 选不选?

    - **不选**: `dp[j]` 保持上一轮 (i.e., 没考虑 n 时的值).
    - **选**: `dp[j-n] + n`.

    取最大: `dp[j] = max(dp[j], dp[j-n] + n)`.

5. **🔑 倒序遍历容量 — 0/1 背包的灵魂 / Iterate capacity backwards**

    一维滚动数组下, **`j` 从 target 倒着到 n**:

    ```cpp
    for (int n : nums) {
        for (int j = target; j >= n; j--) { ... }
    }
    ```

    - **倒序**: 算 `dp[j]` 用 `dp[j-n]`, 此时 `dp[j-n]` 还是"未考虑物品 n" 的旧值 → 物品 n 只被用一次. ✓
    - **正序**: `dp[j-n]` 会先被更新成"已经选了一个 n", 再传给 `dp[j]` → 物品 n 被用了两次 → 变成**完全背包** (0518 / 0322 是这条路).

    > **一句话: 0/1 背包倒序, 完全背包正序**. 这是整个背包家族最重要的一条规则.

6. **另一种 dp 定义: bool dp[j] = "能否凑出 j" / Boolean variant (更贴题意)**

    既然题目只问"能否", 不需要 max 值, 可以直接用 bool. 转移变 `dp[j] = dp[j] || dp[j-n]`. 等价, 选哪种都行.

## Solution

=== "C++"
    === "v1 推荐: 0/1 背包 + 最大价值 (Yang 原版)"
        ```cpp
        class Solution {
        public:
            bool canPartition(vector<int>& nums) {
                int total = accumulate(nums.begin(), nums.end(), 0);
                if (total % 2 != 0) return false;                  // 奇数早返
                int target = total / 2;
                vector<int> dp(target + 1, 0);
                for (int n : nums) {
                    for (int j = target; j >= n; j--) {            // 倒序: 物品只用一次
                        dp[j] = max(dp[j], dp[j - n] + n);
                    }
                }
                return dp[target] == target;
            }
        };
        ```

    === "v2: bool dp 变体"
        ```cpp
        class Solution {
        public:
            bool canPartition(vector<int>& nums) {
                int total = accumulate(nums.begin(), nums.end(), 0);
                if (total % 2 != 0) return false;
                int target = total / 2;
                vector<bool> dp(target + 1, false);
                dp[0] = true;                                      // 空集和为 0
                for (int n : nums) {
                    for (int j = target; j >= n; j--) {
                        dp[j] = dp[j] || dp[j - n];                // 不选 || 选
                    }
                }
                return dp[target];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def canPartition(self, nums: list[int]) -> bool:
            total = sum(nums)
            if total % 2:
                return False
            target = total // 2
            # bool 版更贴 Python 风格. dp[j] = 能否凑出 j
            dp = [False] * (target + 1)
            dp[0] = True
            for n in nums:
                # 倒序: 0/1 背包必须从大到小; range(target, n-1, -1) 等价 C++ for(j=target; j>=n; j--)
                for j in range(target, n - 1, -1):
                    dp[j] = dp[j] or dp[j - n]
            return dp[target]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {boolean}
     */
    var canPartition = function(nums) {
        // reduce 一行求总和: 累加器 sum 初值 0, 每步 sum+x. 等价 C++ accumulate
        const total = nums.reduce((s, x) => s + x, 0);
        if (total % 2 !== 0) return false;
        const target = total / 2;
        const dp = new Array(target + 1).fill(false);
        dp[0] = true;
        for (const n of nums) {
            for (let j = target; j >= n; j--) {                    // 倒序
                dp[j] = dp[j] || dp[j - n];
            }
        }
        return dp[target];
    };
    ```

## Complexity

- **Time**: O(n × target) = O(n × sum/2).
- **Space**: O(target) — 一维滚动数组.

## 相关题目

- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 同 0/1 背包套娃: 求两堆差最小, 等价"凑最接近 total/2 的子集"
- 0494\. Target Sum (待补) — 加减号配方案数, 转化为 0/1 背包计数
- 0474\. Ones and Zeroes (待补) — 二维容量 0/1 背包 (m 个 0 + n 个 1)
- 0322\. Coin Change (待补) — 完全背包 (硬币可重复用) — 对照"正序" vs 本题"倒序"
- 0518\. Coin Change II (待补) — 完全背包计数, 同样是正序
