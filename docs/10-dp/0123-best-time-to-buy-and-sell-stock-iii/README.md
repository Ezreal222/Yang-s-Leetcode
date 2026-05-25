# 0123. Best Time to Buy and Sell Stock III / 买卖股票的最佳时机 III

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, State Machine, Array · 动态规划, 状态机, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给每日股价 `prices`. **最多两次** buy+sell, 求最大利润 (一次都不做返 0).

**中文**: 最多 2 次买卖, 求最大利润.

## Key Insights

1. **🔑 5 状态状态机 / 5-state state machine**

    跟 [0121](../0121-best-time-to-buy-and-sell-stock/README.md) 的 2 状态 ↔ [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) 的 2 状态对照, **0123 需要 5 个状态**, 因为要追踪"目前完成了几次交易":

    | 状态 | 含义 |
    |---|---|
    | `0` | 还没操作 (利润 0, 永远是 0) |
    | `1` | 第一次**买入** 后持有 |
    | `2` | 第一次**卖出** 后 (1 次交易完成) |
    | `3` | 第二次**买入** 后持有 |
    | `4` | 第二次**卖出** 后 (2 次交易完成, **最终答案**) |

    严格按 `0 → 1 → 2 → 3 → 4` 推进 (或在某状态停留), 不会跳级.

2. **转移: 五个状态各自的"卖 / 买 / 不动" / Transitions**

    每个非零状态的转移就是"留在原状态 vs 从上个状态跳过来":

    $$\begin{aligned}
    dp[i][1] &= \max(dp[i-1][1],\ dp[i-1][0] - prices[i]) & \text{第一次买} \\
    dp[i][2] &= \max(dp[i-1][2],\ dp[i-1][1] + prices[i]) & \text{第一次卖} \\
    dp[i][3] &= \max(dp[i-1][3],\ dp[i-1][2] - prices[i]) & \text{第二次买} \\
    dp[i][4] &= \max(dp[i-1][4],\ dp[i-1][3] + prices[i]) & \text{第二次卖}
    \end{aligned}$$

    > **每一行就是一个 0121 / 0122 风格的"卖 or 买"**, 只是连成链. 看到 5 行你可能慌, 其实是同一个模板贴 4 次.

3. **状态 0 永远是 0 → 可省略 / State 0 is always zero**

    `dp[i][0] = 0` 恒成立 (从没操作就是零利润), 所以可以省掉这一维, 直接在转移里把 `dp[i-1][0]` 写成 `0`. Yang 的 v1 保留了 5 维只是更直观, 实际只需要 4 个有意义状态.

4. **初始化: `dp[0][1] = dp[0][3] = -prices[0]` / Base case**

    第 0 天结束时:

    - `dp[0][0] = 0` (没操作)
    - `dp[0][1] = -prices[0]` (今天第一次买入)
    - `dp[0][2] = 0` (今天买完又卖, 利润 0)
    - `dp[0][3] = -prices[0]` (相当于"今天买卖再买", 净 = -prices[0])
    - `dp[0][4] = 0`

    Yang 用 `vector<vector<int>>(n, vector<int>(5, 0))` 初始化所有为 0, 然后单独设 `dp[0][1]` 和 `dp[0][3]` — 其他默认 0 正好对.

5. **🔑 滚动 O(1): 4 个变量 + 顺序级联 / Rolling 4 vars with sequential cascade**

    Yang 的 v2 把 `dp` 压成 4 个变量 (`buy1, sell1, buy2, sell2`), 注意**更新顺序**:

    ```cpp
    buy1  = max(buy1,  -prices[i]);              // 用 prices[i]
    sell1 = max(sell1, buy1 + prices[i]);        // 用 "刚更新的 buy1"
    buy2  = max(buy2,  sell1 - prices[i]);       // 用 "刚更新的 sell1"
    sell2 = max(sell2, buy2 + prices[i]);        // 用 "刚更新的 buy2"
    ```

    **为什么级联安全 — 答案不会偏大?** 看似 "今天买立刻卖"  会双重计入, 但同一天 buy + sell 的利润是 0 (买价 = 卖价), `max(...)` 不会被 0 更新. 严格证明: 即使用了"今天 buy today sell" 当中间状态, 也只是引入了等同于"什么都不做" 的零利润分支, 不影响最大值. ✓

    > 这个**顺序级联** 是 0123/0188 滚动版的标志. 反序写就错了 — 因为后面状态依赖前面更新.

6. **泛化到 0188 k 次交易 / Generalizes to k transactions**

    把 4 个有意义状态扩成 `2k` 个 (`buy_1, sell_1, ..., buy_k, sell_k`), 转移和级联模板**一字不差**. 0123 是 0188 的 `k=2` 特例.

## Solution

=== "C++"
    === "v1 推荐: 5 状态 DP (易读)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int n = prices.size();
                vector<vector<int>> dp(n, vector<int>(5, 0));
                dp[0][1] = -prices[0];                             // 第 0 天第一次买
                dp[0][3] = -prices[0];                             // 等价"今天买卖再买" → 净 -prices[0]
                for (int i = 1; i < n; i++) {
                    dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);  // 第一次买
                    dp[i][2] = max(dp[i - 1][2], dp[i - 1][1] + prices[i]);  // 第一次卖
                    dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]);  // 第二次买
                    dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]);  // 第二次卖 ← 答案
                }
                return dp[n - 1][4];
            }
        };
        ```

    === "v2 推荐: 滚动 O(1) (顺序级联)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int buy1 = -prices[0], sell1 = 0;
                int buy2 = -prices[0], sell2 = 0;
                for (int i = 1; i < (int)prices.size(); i++) {
                    // 顺序级联: 每一行用"刚更新的上一个变量"
                    buy1  = max(buy1,  -prices[i]);
                    sell1 = max(sell1, buy1 + prices[i]);
                    buy2  = max(buy2,  sell1 - prices[i]);
                    sell2 = max(sell2, buy2 + prices[i]);
                }
                return sell2;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, prices: list[int]) -> int:
            # 滚动版, 顺序级联跟 C++ v2 一样
            # buy/sell 都从 -prices[0] / 0 起步, 让 "什么都不做" 利润 = 0
            buy1, sell1 = -prices[0], 0
            buy2, sell2 = -prices[0], 0
            for p in prices[1:]:
                buy1  = max(buy1,  -p)
                sell1 = max(sell1, buy1 + p)
                buy2  = max(buy2,  sell1 - p)
                sell2 = max(sell2, buy2 + p)
            return sell2
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @return {number}
     */
    var maxProfit = function(prices) {
        let buy1 = -prices[0], sell1 = 0;
        let buy2 = -prices[0], sell2 = 0;
        for (let i = 1; i < prices.length; i++) {
            buy1  = Math.max(buy1,  -prices[i]);
            sell1 = Math.max(sell1, buy1 + prices[i]);
            buy2  = Math.max(buy2,  sell1 - prices[i]);
            sell2 = Math.max(sell2, buy2 + prices[i]);
        }
        return sell2;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0121. Best Time to Buy and Sell Stock](../0121-best-time-to-buy-and-sell-stock/README.md) — 1 次交易, 2 状态
- [0122. Best Time to Buy and Sell Stock II](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) — 无限次, 贪心或 2 状态 DP
- [0188. Best Time to Buy and Sell Stock IV](../0188-best-time-to-buy-and-sell-stock-iv/README.md) — **最多 k 次**, 本题泛化 (`k=2` 即本题)
- [0309. Best Time to Buy and Sell Stock with Cooldown](../0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) — 无限次 + 冷冻期, 3 状态
- 0714\. Best Time to Buy and Sell Stock with Transaction Fee (待补) — 无限次 + 手续费
- [§10 DP 思维流程 — 状态机 DP](../topic-dp-thinking-process.md) — 多状态 DP 模板的典型
