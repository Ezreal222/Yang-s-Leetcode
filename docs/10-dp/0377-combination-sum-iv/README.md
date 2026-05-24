# 0377. Combination Sum IV / 组合总和 Ⅳ

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack, Permutation Count · 动态规划, 完全背包, 排列计数
    - **Link**: [LeetCode](https://leetcode.com/problems/combination-sum-iv/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给只含正整数且**互不相同**的 `nums` 和 `target`, 求**和为 `target` 的不同序列数** (顺序不同算不同). 元素可重复用.

**中文**: 给互不相同的正整数 `nums` 和 `target`, 求和为 `target` 的不同**序列**数 (顺序计入, 元素可重复).

## Key Insights

1. **题目名误导: 名字叫"组合", 实际是排列 / Despite the name, it counts permutations**

    LC 把 `[1,1,2]`, `[1,2,1]`, `[2,1,1]` 算**三种**. 这是排列, 不是组合. 看测试用例比看题目名更靠谱.

    > 区分组合 vs 排列, 看"`[a,b]` 和 `[b,a]` 算几种" 这一条就够.

2. **🔑 完全背包 + 外层 j / 内层 nums = 排列数 / Outer capacity, inner items → permutations**

    跟 [0518 Coin Change II](../0518-coin-change-ii/README.md) 的循环顺序对比:

    | 题 | 外层 | 内层 | 算 |
    |---|---|---|---|
    | 0518 | coins | j (正序) | **组合数** |
    | **0377 (本题)** | **j** | **nums** | **排列数** |

    都是完全背包, 转移式都是 `dp[j] += dp[j-n]`. **唯一差别就是循环顺序**.

3. **为什么外 j 就是排列? "按最后加的数" 分类 / Why outer j = permutations**

    `dp[j]` 想象为"凑出 j 的所有序列, 按**最后加的那个数**分类":

    ```
    dp[3] (nums=[1,2,3]) = dp[2] + dp[1] + dp[0]
                          ──┬──   ──┬──   ──┬──
                       最后+1   最后+2   最后+3
                       (2种)    (1种)    (1种)
    ```

    `dp[2]` 里的每条序列, 后面拼一个 "1", 就是一条"凑出 3 且最后加 1" 的序列. 这种"最后一位不同 ⟹ 算不同序列" 的分类天然保留顺序 ⇒ 排列数.

    > 对照 0518: 外 coin 顺序确定了每枚硬币"何时被考虑". coin₂ 加入计算时, coin₁ 早已处理完, 再不会回去和 coin₂ 互换出现顺序 ⇒ 同一组合只算一次 ⇒ 组合数.

4. **`unsigned int` 不是过度防御, 是必需 / `unsigned` is mandatory here**

    跟 [0518](../0518-coin-change-ii/README.md) 不同, 本题 LC 测试里**中间 `dp[j]` 会溢出 32 位有符号 int**, 但**最终答案保证 ≤ INT_MAX**. 用 `unsigned int` (溢出 wrap around, mod 2^32) 或 `long long` (容量够) 都能 work; 用 `int` 在中间步会触发未定义行为, 提交 WA.

    > "答案在 int 内但中间会溢出" 是 LC 计数 DP 的常见坑. 看到测试样例里有 nums 很小 + target 很大 (比如 `nums=[1,2,3]`, target=32) 先用大容量整数试.

5. **`if (j >= n)` 守卫 / Guard against negative index**

    内层 `dp[j-n]` 要 `j >= n` 才合法. Yang 写了 if; 也可以把内层从 `n` 开始而不是 1 (但那样要换成外层固定写法), 当前写法清晰直接.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int combinationSum4(vector<int>& nums, int target) {
            vector<unsigned int> dp(target + 1, 0);                // unsigned 是必需 — 中间会溢出
            dp[0] = 1;                                             // 空序列 = 一种
            for (int j = 1; j <= target; j++) {                    // 外: 容量 → 排列数
                for (int n : nums) {                               // 内: 物品
                    if (j >= n) dp[j] += dp[j - n];
                }
            }
            return dp[target];                                     // unsigned → int 安全 (答案保证 ≤ INT_MAX)
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def combinationSum4(self, nums: list[int], target: int) -> int:
            # Python 整数无大小限制, 不用考虑溢出
            dp = [0] * (target + 1)
            dp[0] = 1
            for j in range(1, target + 1):                         # 外: 容量
                for n in nums:                                     # 内: 物品
                    if j >= n:
                        dp[j] += dp[j - n]
            return dp[target]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number}
     */
    var combinationSum4 = function(nums, target) {
        // JS Number 是 64-bit float, 整数精度到 2^53, 中间溢出不会发生在本题范围
        const dp = new Array(target + 1).fill(0);
        dp[0] = 1;
        for (let j = 1; j <= target; j++) {
            for (const n of nums) {
                if (j >= n) dp[j] += dp[j - n];
            }
        }
        return dp[target];
    };
    ```

## Complexity

- **Time**: O(target × n), n = `nums.size()`.
- **Space**: O(target).

## 相关题目

- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 完全背包**组合数** (外 coin/内 j), 跟本题就是循环顺序之差
- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 0/1 背包母题, 对比"倒序 j"
- [0494. Target Sum](../0494-target-sum/README.md) — 0/1 背包计数
- 0322\. Coin Change (待补) — 完全背包求**最少硬币数** (min 取代 +=)
- 0070\. Climbing Stairs — 把"步长 1 或 2" 当 nums, 就是本题特例: 排列数 = 走法数 → [Climbing Stairs](../0070-climbing-stairs/README.md) 的完全背包视角
- 0279\. Perfect Squares (待补) — 完全背包: 平方数当物品, 求最少个数
- 0139\. Word Break (待补) — 完全背包变体 (字符串拼接)
