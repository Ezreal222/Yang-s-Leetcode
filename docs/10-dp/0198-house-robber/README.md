# 0198. House Robber / 打家劫舍

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP · 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/house-robber/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 一排房子, 每间藏钱 `nums[i]`. 偷相邻两间会触发警报, 求**最多能偷多少**.

**中文**: 一排房子 `nums`, 不能偷相邻, 求最多能偷的金额.

## Key Insights

1. **"选/不选" 思维 + 相邻约束 / Pick/skip with adjacency constraint**

    每间房只有两种处理: 偷, 或不偷. "偷" 受**相邻约束** — 不能再偷上一间.

    > 这是线性 DP 最常见的模型 — "每个元素二元选择 + 选了就锁死相邻". 跟 [§10 DP 思维流程](../topic-dp-thinking-process.md) 里的"选/不选" 模式吻合.

2. **状态: `dp[i] = 偷到第 i 间时的最大金额` / dp[i] = max so far up to house i**

    "**到** 第 i 间" 而不是"**偷** 第 i 间" — 包含"决定不偷第 i 间" 的情况, 不强制选当前.

    > 跟 [0300 LIS (待补)](#) 的"以 i 结尾" 强制选有本质区别. **状态定义的强弱直接决定转移**.

3. **🔑 用"最后一步" 思维推转移 / Last-step thinking**

    到达 `dp[i]` 的最后一步只有两种:

    - **偷第 i 间**: 上一间必须没偷 → `dp[i-2] + nums[i]`.
    - **不偷第 i 间**: 没新进账 → `dp[i-1]` 维持.

    取最大:

    $$dp[i] = \max(dp[i-1],\ dp[i-2] + nums[i])$$

4. **初始化要小心: `dp[1] = max(nums[0], nums[1])` / Tricky base case**

    `dp[0] = nums[0]` 显然. **`dp[1]` 不是 `nums[1]`** — 因为"到第 1 间" 可以选择只偷第 0 间. 必须取 max.

    > 漏写 max 是新手最常 bug. 想"`dp[i]` 是到 i 的最优, 包括不偷 i 的情况", 这一致性自然推出 `dp[1] = max(nums[0], nums[1])`.

5. **滚动变量 O(1) 空间 / Rolling pair**

    `dp[i]` 只依赖 `dp[i-1]` 和 `dp[i-2]` → 两个变量足够. 同 [0509](../0509-fibonacci-number/README.md) / [0070](../0070-climbing-stairs/README.md) / [0746](../0746-min-cost-climbing-stairs/README.md) 套路.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int rob(vector<int>& nums) {
            int n = nums.size();
            if (n == 1) return nums[0];                            // 防越界, n=1 时 dp[1] 无意义
            int dp[2];
            dp[0] = nums[0];
            dp[1] = max(nums[0], nums[1]);                         // 关键: 不是 nums[1]
            for (int i = 2; i < n; i++) {
                int cur = max(dp[1], dp[0] + nums[i]);             // 不偷 vs 偷
                dp[0] = dp[1];                                     // 滚动
                dp[1] = cur;
            }
            return dp[1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def rob(self, nums: list[int]) -> int:
            if len(nums) == 1:
                return nums[0]
            # 元组解包一行滚动, 跟 0070/0509 同套路
            # prev2, prev1 分别对应 dp[i-2], dp[i-1]
            prev2, prev1 = nums[0], max(nums[0], nums[1])
            for x in nums[2:]:
                prev2, prev1 = prev1, max(prev1, prev2 + x)
            return prev1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var rob = function(nums) {
        if (nums.length === 1) return nums[0];
        let [prev2, prev1] = [nums[0], Math.max(nums[0], nums[1])];
        for (let i = 2; i < nums.length; i++) {
            [prev2, prev1] = [prev1, Math.max(prev1, prev2 + nums[i])];
        }
        return prev1;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1) — 滚动变量.

## 相关题目

- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 同滚动 DP 结构, 计数版
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 同 `dp[i] = f(dp[i-1], dp[i-2])` 模板
- 0213\. House Robber II (待补) — 环形版, 拆成两条线性跑 0198
- 0337\. House Robber III (待补) — 树形版, 树形 DP + 三状态
- 0740\. Delete and Earn (待补) — 转化版: 按值分桶 + 0198
- [§10 DP 思维流程 — 选/不选 + 最后一步](../topic-dp-thinking-process.md) — 本题就是这两个模式的合体
