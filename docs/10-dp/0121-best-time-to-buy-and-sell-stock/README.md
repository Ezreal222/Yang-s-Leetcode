# 0121. Best Time to Buy and Sell Stock / 买卖股票的最佳时机

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, State Machine, Greedy, Array · 动态规划, 状态机, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给每日股价 `prices`. 只能**买一次卖一次** (先买后卖), 求最大利润. 一次交易都不做返 0.

**中文**: 只允许一次买卖, 求最大利润.

## Key Insights

1. **两种解法: 贪心一次扫 vs 状态机 DP / Greedy single-pass vs state machine DP**

    | 解法 | 思路 | 优劣 |
    |---|---|---|
    | **贪心 (v1)** | 维护"目前最低买入价", 每天算当前卖出能赚多少, 更新答案 | 短 + O(1) 空间, 但**不可推广** |
    | **状态机 DP (v2)** | 两个状态 (持有/不持有), 显式记录两种最优 | 长一点, 但**是整个股票系列 (0122 / 0123 / 0188 / 0309 / 0714) 的种子模板** |

    > 0121 是状态机 DP 的入门. 后续 0122 / 0123 / 0188 / 0309 / 0714 全是这个模板**加状态轴** (交易次数, 冷冻期, 手续费等). **学会 DP 版收益大**.

2. **🔑 状态机 DP 核心: 两个状态 + "最后一步" 推转移 / Two states, last-step thinking**

    每天结束时只有两种身份:

    | 状态 | 含义 |
    |---|---|
    | `dp[i][0]` | 第 i 天结束时**不持有** 股票 (手里没货) |
    | `dp[i][1]` | 第 i 天结束时**持有** 股票 (手里有一股) |

    用[最后一步思维](../topic-dp-thinking-process.md): 今天怎么变成这个状态?

    - **不持有 (`dp[i][0]`)** 的最后一步:
        - 昨天就不持有, 今天**不动** → `dp[i-1][0]`
        - 昨天持有, 今天**卖出** → `dp[i-1][1] + prices[i]` (收入 prices[i])
        - `dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])`

    - **持有 (`dp[i][1]`)** 的最后一步:
        - 昨天就持有, 今天**不动** → `dp[i-1][1]`
        - 昨天不持有, 今天**买入** → `-prices[i]` (花 prices[i])
        - `dp[i][1] = max(dp[i-1][1], -prices[i])`

3. **🔑 为什么"买入" 那行是 `-prices[i]` 而不是 `dp[i-1][0] - prices[i]` / Why hardcoded -prices[i] for buying**

    这是 0121 跟后面 [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) 最大的差别. **0121 只允许一次交易** ⇒ 买入之前必然**还没卖过**任何东西 ⇒ 利润必为 0 ⇒ 买入花费就是 `-prices[i]`.

    若改成 `dp[i-1][0] - prices[i]` (用昨天不持有的利润减去今天买价), 就允许"卖了再买" → 退化成 0122 (多次交易). **这一字之差就是 0121 vs 0122**.

    > 后面 0122/0123 等只要把 `-prices[i]` 改成 `dp[i-1][0] - prices[i]` 或加更多状态轴就能升级. 状态机 DP 的统一性在这里展现.

4. **初始化: 第 0 天 / Base case**

    - `dp[0][0] = 0` (第 0 天结束没持有, 没操作 → 利润 0)
    - `dp[0][1] = -prices[0]` (第 0 天结束持有, 必是今天买的 → 利润 -prices[0])

5. **答案在 `dp[n-1][0]`, 不在 `dp[n-1][1]` / Answer is "not holding" at end**

    最后一天若还持有, 说明买了没卖 → 利润亏一笔买价. 最优解最后必然不持有.

6. **可压到 O(1) 空间 / Rolling pair**

    `dp[i][*]` 只依赖 `dp[i-1][*]` → 两个变量足够. 见 [§10 DP 思维流程 — 滚动数组](../topic-dp-thinking-process.md).

## Solution

=== "C++"
    === "v2 推荐: 状态机 DP (可推广)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int n = prices.size();
                vector<vector<int>> dp(n, vector<int>(2));
                dp[0][0] = 0;
                dp[0][1] = -prices[0];
                for (int i = 1; i < n; i++) {
                    dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);   // 卖 or 不动
                    dp[i][1] = max(dp[i - 1][1], -prices[i]);                 // 买 or 不动 (仅一次交易!)
                }
                return dp[n - 1][0];
            }
        };
        ```

    === "v1: 贪心一次扫 (O(1) 空间)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int minPrice = prices[0], maxProfit = 0;
                for (int p : prices) {
                    maxProfit = max(maxProfit, p - minPrice);
                    minPrice = min(minPrice, p);
                }
                return maxProfit;
            }
        };
        ```

    === "v3: 状态机 DP + 滚动 O(1)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int hold = -prices[0], cash = 0;                    // 持有 / 不持有
                for (int i = 1; i < prices.size(); i++) {
                    cash = max(cash, hold + prices[i]);             // 卖
                    hold = max(hold, -prices[i]);                   // 买 (仅一次)
                }
                return cash;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, prices: list[int]) -> int:
            # 滚动版状态机 DP, 跟 C++ v3 一一对应
            # hold = 持有股票的最大利润, cash = 不持有的最大利润
            hold, cash = -prices[0], 0
            for p in prices[1:]:
                cash = max(cash, hold + p)                          # 卖
                hold = max(hold, -p)                                # 买 (注意是 -p 不是 cash-p)
            return cash
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @return {number}
     */
    var maxProfit = function(prices) {
        let hold = -prices[0], cash = 0;
        for (let i = 1; i < prices.length; i++) {
            cash = Math.max(cash, hold + prices[i]);                // 卖
            hold = Math.max(hold, -prices[i]);                      // 买 (仅一次)
        }
        return cash;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v2) / O(1) (v1, v3).

## 相关题目

- [0122. Best Time to Buy and Sell Stock II](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) — **无限次交易**, `dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])` (在 §09, 可贪心)
- [0123. Best Time to Buy and Sell Stock III](../0123-best-time-to-buy-and-sell-stock-iii/README.md) — **最多 2 次交易**, 加交易次数轴 (5 状态 DP)
- [0188. Best Time to Buy and Sell Stock IV](../0188-best-time-to-buy-and-sell-stock-iv/README.md) — **最多 k 次**, 0123 的泛化
- [0309. Best Time to Buy and Sell Stock with Cooldown](../0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) — **加冷冻期** 一天, 3 状态
- 0714\. Best Time to Buy and Sell Stock with Transaction Fee (待补) — **加手续费**
- [§10 DP 思维流程 — 状态机 DP](../topic-dp-thinking-process.md) — 本题是入门, 多状态转移的典型例子
