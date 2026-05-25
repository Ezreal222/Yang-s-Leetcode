# 0714. Best Time to Buy and Sell Stock with Transaction Fee / 买卖股票的最佳时机含手续费

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, State Machine, Greedy, Array · 动态规划, 状态机, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 每日股价 `prices` + 每笔交易手续费 `fee`. 无限次买卖, 每**完成一次**交易扣 `fee`. 求最大利润.

**中文**: 无限次买卖, 每完成一次交易扣 `fee`. 求最大利润.

## Key Insights

1. **🔑 [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) 状态机 DP **直接加一个 -fee** / Drop-in fee modification**

    跟 0122 完全同骨架, **唯一改动**: 卖出那行减一个 `fee` (扣手续费).

    | 行 | [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) (无 fee) | **0714 (本题)** |
    |---|---|---|
    | 卖出 | `dp[i][1] = max(dp[i-1][1], dp[i-1][0] + prices[i])` | `dp[i][1] = max(dp[i-1][1], dp[i-1][0] + prices[i] - fee)` |
    | 买入 | `dp[i][0] = max(dp[i-1][0], dp[i-1][1] - prices[i])` | **完全一样** |

    > 把 fee 算在卖出 (Yang 的写法) 还是买入都对, 只要**一次完整交易扣一次** 就行. 卖出扣更自然 — 跟 "完成一次交易" 同步.

2. **手续费让贪心 (0122 累加正差) 失效 / Fee kills the greedy approach**

    0122 的贪心 "每个正差都收" 不再对: 小幅波动 (差额 < fee) 反而亏. 必须 DP 才能正确权衡"该不该交易".

    > 加任何"代价" 类约束 (cooldown / fee / k 次) 都让贪心失效, 必须升级 DP. 这是整个股票系列的统一规律.

3. **状态/转移/初始化跟 0122 一字不差 / Otherwise identical to 0122**

    - 状态: `dp[i][0]` = 第 i 天持有, `dp[i][1]` = 第 i 天不持有.
    - 初始: `dp[0][0] = -prices[0]`, `dp[0][1] = 0`.
    - 答案: `dp[n-1][1]`.
    - 可滚动到 O(1) (`hold` + `cash` 两个变量).

4. **fee 不能扣两次 / Don't double-charge**

    转移里只有"卖出" 那一行扣 `-fee`. 买入不扣. 写成两边都扣就把利润减半了, 错.

5. **跟 [0309 Cooldown](../0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) 的对比 / vs Cooldown**

    | | 0122 | **0714 (fee)** | [0309 (cooldown)](../0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) |
    |---|---|---|---|
    | 约束 | 无 | 完成交易扣 fee | 卖出次日不能买 |
    | 状态数 | 2 | **2** (+ 转移加 -fee) | 3 (拆"不持有") |
    | 改 0122 哪里? | — | **卖出行 -fee** | 加状态, 重写转移 |

    > fee 是改约束**最轻**的方式 — 状态不变, 加个常数. cooldown 改约束更重, 状态都得拆.

## Solution

=== "C++"
    === "v1 推荐: 二维 DP (易读)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices, int fee) {
                int n = prices.size();
                vector<vector<int>> dp(n, vector<int>(2, 0));
                dp[0][0] = -prices[0];                                          // 持有: 今天买
                for (int i = 1; i < n; i++) {
                    dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);              // 买
                    dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i] - fee);        // 卖 (扣 fee)
                }
                return dp[n - 1][1];
            }
        };
        ```

    === "v2: 滚动 O(1)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices, int fee) {
                int hold = -prices[0], cash = 0;
                for (int i = 1; i < (int)prices.size(); i++) {
                    int newHold = max(hold, cash - prices[i]);
                    int newCash = max(cash, hold + prices[i] - fee);
                    hold = newHold; cash = newCash;
                }
                return cash;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, prices: list[int], fee: int) -> int:
            # 滚动 O(1) 版, 跟 0122 DP 一字之差 (卖时扣 fee)
            hold, cash = -prices[0], 0
            for p in prices[1:]:
                hold, cash = max(hold, cash - p), max(cash, hold + p - fee)
            return cash
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @param {number} fee
     * @return {number}
     */
    var maxProfit = function(prices, fee) {
        let hold = -prices[0], cash = 0;
        for (let i = 1; i < prices.length; i++) {
            const newHold = Math.max(hold, cash - prices[i]);
            const newCash = Math.max(cash, hold + prices[i] - fee);
            hold = newHold; cash = newCash;
        }
        return cash;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0121. Best Time to Buy and Sell Stock](../0121-best-time-to-buy-and-sell-stock/README.md) — 1 次, 2 状态
- [0122. Best Time to Buy and Sell Stock II](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) — 无限次, 无 fee (本题 fee=0 的极限)
- [0123. Best Time to Buy and Sell Stock III](../0123-best-time-to-buy-and-sell-stock-iii/README.md) — 最多 2 次
- [0188. Best Time to Buy and Sell Stock IV](../0188-best-time-to-buy-and-sell-stock-iv/README.md) — 最多 k 次
- [0309. Best Time to Buy and Sell Stock with Cooldown](../0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) — 无限次 + 冷冻期, 3 状态
- [§10 DP 思维流程 — 状态机 DP](../topic-dp-thinking-process.md) — 加 fee 是"加约束最轻" 的例子, 不动状态只改转移
