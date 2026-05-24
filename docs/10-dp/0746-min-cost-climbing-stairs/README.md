# 0746. Min Cost Climbing Stairs / 使用最小花费爬楼梯

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, Array · 动态规划, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/min-cost-climbing-stairs/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 数组 `cost[i]` 是踩在第 i 个台阶要付的费. 可从 index `0` 或 `1` 起步 (起步本身免费), 每步上 1 或 2 阶. 求到达**顶端** (index `n`, 不在数组里) 的最小总费用.

**中文**: `cost[i]` 是第 i 阶的费用, 起点可选 0 或 1 (免费), 每步上 1 或 2, 求到第 `n` 阶 (越过最后一阶) 的最小总花费.

## Key Insights

1. **状态定义: dp[i] = 到达第 i 阶的最小花费 (不含 cost[i]) / dp[i] = min cost to reach stair i**

    "到达" 是关键 — 还没踩这阶, 所以 `dp[i]` 不含 `cost[i]`. 转移时**离开**前一阶才付费:

    $$dp[i] = \min(dp[i-1] + cost[i-1],\ dp[i-2] + cost[i-2])$$

    > "费用什么时候付" 是 DP 状态设计的常见陷阱. 题目说"踩到要付", 那"还没踩" 的状态最干净.

2. **初始化: `dp[0] = dp[1] = 0` / Free to start at either 0 or 1**

    题目允许从 0 或 1 起步且**起步免费**, 所以这两个起点状态都是 0. 没有"从更前面来" 的转移, 直接赋值.

    > 跟 [0070 爬楼梯](../0070-climbing-stairs/README.md) 的 `dp[1]=1, dp[2]=2` 对比 — 那题是**计数** (一种方法), 这题是**优化** (零花费). 同框架, 不同语义.

3. **顶端是 index `n`, 不是 `n-1` / Top is past the array**

    `cost.size() = n` 意味台阶编号 `0..n-1`, **顶端是 n** (虚拟一阶, 不收费). 所以 dp 数组开 `n+1`, 返回 `dp[n]`. 这点最容易翻车.

4. **滚动变量 O(1) 空间 / Rolling pair**

    同 [0509](../0509-fibonacci-number/README.md) / [0070](../0070-climbing-stairs/README.md), 只依赖前两个状态. 留在 v2 演示.

## Solution

=== "C++"
    === "v1: 完整 dp 数组 (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int minCostClimbingStairs(vector<int>& cost) {
                int n = cost.size();
                vector<int> dp(n + 1, 0);                          // dp[0]=dp[1]=0 (起步免费)
                for (int i = 2; i <= n; i++) {
                    dp[i] = min(dp[i - 1] + cost[i - 1],           // 从 i-1 阶离开 (付 cost[i-1])
                                dp[i - 2] + cost[i - 2]);          // 从 i-2 阶离开 (付 cost[i-2])
                }
                return dp[n];
            }
        };
        ```

    === "v2: 滚动变量 O(1)"
        ```cpp
        class Solution {
        public:
            int minCostClimbingStairs(vector<int>& cost) {
                int n = cost.size();
                int prev2 = 0, prev1 = 0;                          // dp[i-2], dp[i-1]
                for (int i = 2; i <= n; i++) {
                    int cur = min(prev1 + cost[i - 1], prev2 + cost[i - 2]);
                    prev2 = prev1;
                    prev1 = cur;
                }
                return prev1;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def minCostClimbingStairs(self, cost: list[int]) -> int:
            # 元组解包滚动, 跟 0070 / 0509 同一招
            # prev2, prev1 = dp[i-2], dp[i-1]
            prev2, prev1 = 0, 0
            for i in range(2, len(cost) + 1):
                prev2, prev1 = prev1, min(prev1 + cost[i - 1], prev2 + cost[i - 2])
            return prev1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} cost
     * @return {number}
     */
    var minCostClimbingStairs = function(cost) {
        const n = cost.length;
        let [prev2, prev1] = [0, 0];                               // dp[i-2], dp[i-1]
        for (let i = 2; i <= n; i++) {
            // Math.min 接散参; 想传数组得用 ...spread, 但本题只两项直接列
            [prev2, prev1] = [prev1, Math.min(prev1 + cost[i - 1], prev2 + cost[i - 2])];
        }
        return prev1;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 计数版, 同递推结构
- [0509. Fibonacci Number](../0509-fibonacci-number/README.md) — 滚动 DP 母题
- [0198. House Robber](../0198-house-robber/README.md) — 同模板: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`, 选不选当前元素
- 0213\. House Robber II (待补) — 环形版, 拆两次跑 0198
