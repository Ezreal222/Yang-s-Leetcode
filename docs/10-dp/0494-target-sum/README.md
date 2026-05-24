# 0494. Target Sum / 目标和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, 0/1 Knapsack, Counting, Backtracking · 动态规划, 01 背包, 计数, 回溯
    - **Link**: [LeetCode](https://leetcode.com/problems/target-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给非负整数数组 `nums` 和整数 `target`. 给每个数前面加 `+` 或 `−` 号, 求**让总和恰为 `target` 的方案数**.

**中文**: 给 `nums` 每个数前加 `+/-` 号, 求总和恰为 `target` 的方案数.

## Key Insights

1. **核心转化: 给 ± 号 ⟺ 把数分成"正堆 + 负堆", 求正堆和 = `(total + target) / 2` / Sign assignment ⟺ split into "+" and "−" piles**

    设正堆和 `pos`, 负堆和 `neg`. 联立:

    $$\begin{cases} pos + neg = total \\ pos - neg = target \end{cases} \Rightarrow pos = \frac{total + target}{2}$$

    问题变成: **从 nums 里挑子集, 让子集和 = `(total + target) / 2`, 求方案数**. → 0/1 背包计数问题.

    > 这是"分两堆 ± 号" 类题的标准转化, 跟 [1049](../1049-last-stone-weight-ii/README.md) 几乎一样, 只是那题求最小差, 这题求方案数.

2. **两个不可能的早返 / Two impossibility checks**

    - **`abs(target) > total`**: 全 + 也凑不到 target, 全 − 也凑不到 −target. 直接 0.
    - **`(total + target) % 2 != 0`**: `pos` 必须是整数. 奇数无法被 2 整除 → 0.

    > 这两个 check 应该放在算 `bagSize` **之前**, 避免 `bagSize` 出现负数 (Yang 原代码顺序略弱, 不出错但概念不干净).

3. **0/1 背包计数模板: `dp[j] += dp[j-n]` / Counting variant**

    跟 [0416](../0416-partition-equal-subset-sum/README.md) 的判定/最大化版对照:

    | 题 | 状态 | 转移 | 初始 |
    |---|---|---|---|
    | 0416 (判定) | `bool dp[j]` | `dp[j] \|\|= dp[j-n]` | `dp[0] = true` |
    | 0416 (求最大装) | `int dp[j]` | `dp[j] = max(dp[j], dp[j-n] + n)` | `dp[0] = 0` |
    | **0494 (计数)** | `int dp[j]` | `dp[j] += dp[j-n]` | `dp[0] = 1` |

    `dp[j]` = 凑出和为 `j` 的方案数. 转移 = "**不选 n** 的方案数 (`dp[j]` 旧值) + **选 n** 的方案数 (`dp[j-n]`)".

4. **`dp[0] = 1` 是关键 / Base case: one way to make 0**

    "什么都不选" 也算一种方案 → `dp[0] = 1`. 若设 0, 整个 dp 都是 0.

    > 这跟 [0096](../0096-unique-binary-search-trees/README.md) 的"空树算一种" 同理 — **计数 DP 的空集都算 1**.

5. **倒序 j: 0/1 背包标配 / Iterate j backwards**

    跟 [0416](../0416-partition-equal-subset-sum/README.md) 同理 — 每个数最多被用一次, 必须倒序 `for (int j = bagSize; j >= n; j--)`. 正序就退化成完全背包 (每个数可以被多次"使用" 在不同 +/- 配置里, 跟题意不符).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findTargetSumWays(vector<int>& nums, int target) {
            int total = accumulate(nums.begin(), nums.end(), 0);
            // 两个早返先于 bagSize 计算更干净
            if (abs(target) > total) return 0;
            if ((total + target) % 2 != 0) return 0;
            int bagSize = (total + target) / 2;

            vector<int> dp(bagSize + 1, 0);
            dp[0] = 1;                                             // 空集算一种方案
            for (int n : nums) {
                for (int j = bagSize; j >= n; j--) {               // 倒序: 0/1 背包
                    dp[j] += dp[j - n];                            // 不选 + 选, 方案数累加
                }
            }
            return dp[bagSize];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findTargetSumWays(self, nums: list[int], target: int) -> int:
            total = sum(nums)
            if abs(target) > total or (total + target) % 2:
                return 0
            bag = (total + target) // 2
            dp = [0] * (bag + 1)
            dp[0] = 1                                              # 计数 DP 的空集 = 1
            for n in nums:
                # range(bag, n-1, -1) = C++ for(j=bag; j>=n; j--), 0/1 背包倒序
                for j in range(bag, n - 1, -1):
                    dp[j] += dp[j - n]
            return dp[bag]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number}
     */
    var findTargetSumWays = function(nums, target) {
        const total = nums.reduce((s, x) => s + x, 0);
        if (Math.abs(target) > total || (total + target) % 2 !== 0) return 0;
        const bag = (total + target) / 2;
        const dp = new Array(bag + 1).fill(0);
        dp[0] = 1;
        for (const n of nums) {
            for (let j = bag; j >= n; j--) {
                dp[j] += dp[j - n];
            }
        }
        return dp[bag];
    };
    ```

## Complexity

- **Time**: O(n × (total + target)/2).
- **Space**: O((total + target)/2).

## 相关题目

- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 同 0/1 背包, 求"能否凑到 target"
- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 同 "分 ± 号" 模型, 求最小差
- [0096. Unique Binary Search Trees](../0096-unique-binary-search-trees/README.md) — 同款"空集算一种" 的计数 DP 初始化
- [0474. Ones and Zeroes](../0474-ones-and-zeroes/README.md) — 二维容量 0/1 背包
- 0518\. Coin Change II (待补) — 完全背包计数 (正序 j) — 对比本题"倒序 j" 的 0/1 计数
