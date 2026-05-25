# 0309. Best Time to Buy and Sell Stock with Cooldown / 最佳买卖股票时机含冷冻期

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, State Machine, Array · 动态规划, 状态机, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 每日股价 `prices`. **不限交易次数**, 但**卖出后下一天不能买** (冷冻 1 天). 求最大利润.

**中文**: 无限次交易, 但卖出当天的**次日**不能买. 求最大利润.

## Key Insights

1. **🔑 冷冻期 → 多一个状态 / Cooldown introduces a third state**

    [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) 的"持有 / 不持有" 二态不够用 — "不持有" 现在要分**两种**: 卖完冷冻中 vs 已经过完冷冻可以买. 拆成 3 个状态:

    | 状态 | 含义 | 可以做什么 |
    |---|---|---|
    | `0` 持有 | 手里有股 | 卖 (→ 状态 2) 或不动 (→ 自己) |
    | `1` 不持有, **冷冻已过** | 手里没股, 可以买 | 买 (→ 状态 0) 或不动 (→ 自己) |
    | `2` **今天刚卖出** (冷冻中) | 手里没股, **次日不能买** | 唯一去路: 明天进入状态 1 |

    > 加新约束的 DP, 先想"原状态够不够". 不够就**拆状态**, 别硬塞额外的轴.

2. **转移: 三行 / Three transitions**

    $$\begin{aligned}
    dp[i][0] &= \max(dp[i-1][0],\ dp[i-1][1] - prices[i]) & \text{持有: 不动 / 今天从 "冷冻已过" 买} \\
    dp[i][1] &= \max(dp[i-1][1],\ dp[i-1][2]) & \text{冷冻已过: 不动 / 昨天刚卖完今天就过冷冻} \\
    dp[i][2] &= dp[i-1][0] + prices[i] & \text{今天卖出: 从持有变冷冻}
    \end{aligned}$$

    > 注意"卖出" 那行**没有 max** — 因为定义就是"今天才卖出", 必然从持有来.

3. **🔑 买入的来源是 "状态 1", 不是 "状态 2" / Buy only from non-cooldown idle**

    新手最容易写错 `dp[i][0] = max(dp[i-1][0], dp[i-1][2] - prices[i])` — 那相当于"昨天卖完今天就买", 违反冷冻规则. 必须从 `dp[i-1][1]` 买 (昨天非冷冻 → 今天可买). Yang 写对了.

    > 状态拆得对, 转移才能对. 这是"加约束 DP" 的灵魂.

4. **初始化: `dp[0][0] = -prices[0]`, 其余 0 / Base case**

    - `dp[0][0] = -prices[0]` (第 0 天买入)
    - `dp[0][1] = 0` (没操作, 也算"冷冻已过")
    - `dp[0][2] = 0` (没操作, 不算冷冻; 设 0 不影响)

5. **答案 `max(dp[n-1][1], dp[n-1][2])` / Answer is max of both non-holding states**

    最后必须不持有 (否则亏一笔买价). 两种"不持有" 都可能是终态, 取 max.

6. **可滚动到 O(1) / Rolling 3 vars**

    `dp[i][*]` 只依赖 `dp[i-1][*]` → 3 个变量足够. 注意更新顺序: **算完所有 new_* 再统一赋值**, 别像 [0123 滚动](../0123-best-time-to-buy-and-sell-stock-iii/README.md) 那样级联 — 这里依赖 `dp[i-1][2]` (旧), 先更新它就错.

## Solution

=== "C++"
    === "v1 推荐: 二维 DP (易读)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int n = prices.size();
                vector<vector<int>> dp(n, vector<int>(3, 0));
                dp[0][0] = -prices[0];                                          // 持有: 今天买
                // dp[0][1] = dp[0][2] = 0 (default)
                for (int i = 1; i < n; i++) {
                    dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);     // 持有
                    dp[i][1] = max(dp[i - 1][1], dp[i - 1][2]);                 // 冷冻已过
                    dp[i][2] = dp[i - 1][0] + prices[i];                        // 今天卖
                }
                return max(dp[n - 1][1], dp[n - 1][2]);                         // 终态必不持有
            }
        };
        ```

    === "v2: 滚动 O(1) (注意: 不能级联!)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int hold = -prices[0], rest = 0, sold = 0;                      // 状态 0/1/2
                for (int i = 1; i < (int)prices.size(); i++) {
                    int newHold = max(hold, rest - prices[i]);
                    int newRest = max(rest, sold);
                    int newSold = hold + prices[i];
                    hold = newHold; rest = newRest; sold = newSold;             // 一起赋值, 不级联
                }
                return max(rest, sold);
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, prices: list[int]) -> int:
            # 三状态滚动版. 跟 0123 不同, 这里不能级联 — 必须三个 new_* 全算完再赋值
            # 因为 new_rest 依赖 sold (旧), 先更新 sold 就用错了
            hold, rest, sold = -prices[0], 0, 0
            for p in prices[1:]:
                hold, rest, sold = (
                    max(hold, rest - p),                                       # 持有
                    max(rest, sold),                                           # 冷冻已过
                    hold + p,                                                  # 今天卖
                )
            return max(rest, sold)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @return {number}
     */
    var maxProfit = function(prices) {
        let hold = -prices[0], rest = 0, sold = 0;
        for (let i = 1; i < prices.length; i++) {
            // 用临时变量, 不级联
            const newHold = Math.max(hold, rest - prices[i]);
            const newRest = Math.max(rest, sold);
            const newSold = hold + prices[i];
            hold = newHold; rest = newRest; sold = newSold;
        }
        return Math.max(rest, sold);
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0121. Best Time to Buy and Sell Stock](../0121-best-time-to-buy-and-sell-stock/README.md) — 1 次, 2 状态
- [0122. Best Time to Buy and Sell Stock II](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) — 无限次, 无冷冻 (本题 cooldown=0 的极限)
- [0123. Best Time to Buy and Sell Stock III](../0123-best-time-to-buy-and-sell-stock-iii/README.md) — 最多 2 次
- [0188. Best Time to Buy and Sell Stock IV](../0188-best-time-to-buy-and-sell-stock-iv/README.md) — 最多 k 次
- [0714. Best Time to Buy and Sell Stock with Transaction Fee](../0714-best-time-to-buy-and-sell-stock-with-transaction-fee/README.md) — 无限次 + 手续费, 2 状态 + 卖时扣 fee
- [§10 DP 思维流程 — 状态机 DP](../topic-dp-thinking-process.md) — 加约束 → 拆状态的典型例
