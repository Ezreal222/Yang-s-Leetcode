# 0070. Climbing Stairs / 爬楼梯

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, Math · 动态规划, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/climbing-stairs/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 楼梯有 `n` 阶. 每次能爬 1 或 2 阶. 多少种不同方法爬到顶?

**中文**: 楼梯有 `n` 阶, 每次爬 1 或 2 阶, 问有多少种爬法到达第 `n` 阶.

## Key Insights

1. **状态拆解 → 跟斐波那契一模一样 / Recurrence is just Fibonacci**

    到达第 `i` 阶的方式只有两种来源:

    - 从第 `i-1` 阶迈 1 步上来 → `dp[i-1]` 种
    - 从第 `i-2` 阶迈 2 步上来 → `dp[i-2]` 种

    两类方案不重不漏 (最后一步的步长不同) ⇒ `dp[i] = dp[i-1] + dp[i-2]`.

    > 凡是"最后一步只有有限种选择" 的计数题, 都按"最后一步分类" 列转移. 这是 DP 入门最重要的思维.

2. **初始化跟 [0509](../0509-fibonacci-number/README.md) 不同 / Different base case from Fibonacci**

    本题 `dp[1] = 1`, `dp[2] = 2` (而非 0509 的 `dp[0]=0, dp[1]=1`). 因为题目从第 1 阶开始计数:

    - 到 1 阶: 1 种 (爬 1)
    - 到 2 阶: 2 种 (爬 1+1 或 爬 2)

    > 同样的递推, 初值决定数列起点. 别照抄 0509.

3. **滚动变量 O(1) 空间 / Rolling pair**

    同 [0509](../0509-fibonacci-number/README.md) 思路, 只依赖前两个状态 → 两个变量足够.

4. **进阶: 一次最多爬 m 阶 / Generalization (完全背包雏形)**

    若每步能爬 `1..m` 阶, 则 `dp[i] = sum(dp[i-j] for j in 1..m)`. 这是**完全背包**的特例 — 物品 (步长) 可重复选, 容量 (台阶) 要恰好填满. 留到完全背包统一处理.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int climbStairs(int n) {
            if (n <= 2) return n;
            int dp[3];
            dp[1] = 1;
            dp[2] = 2;
            for (int i = 3; i <= n; i++) {
                int sum = dp[1] + dp[2];
                dp[1] = dp[2];                                     // 滚动: 旧 dp[2] 变 dp[i-2]
                dp[2] = sum;
            }
            return dp[2];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def climbStairs(self, n: int) -> int:
            if n <= 2:
                return n
            # 元组解包一行滚动, 跟 0509 一样
            a, b = 1, 2                                            # dp[1], dp[2]
            for _ in range(3, n + 1):
                a, b = b, a + b
            return b
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var climbStairs = function(n) {
        if (n <= 2) return n;
        let [a, b] = [1, 2];                                       // dp[1], dp[2]
        for (let i = 3; i <= n; i++) {
            [a, b] = [b, a + b];
        }
        return b;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 相关题目

- [0509. Fibonacci Number](../0509-fibonacci-number/README.md) — 同递推, 不同初值
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 加权版, `dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2])`
- 1137\. N-th Tribonacci Number (待补) — 三步滚动
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 完全背包视角: 把"步长" 当物品, 排列数即走法数
