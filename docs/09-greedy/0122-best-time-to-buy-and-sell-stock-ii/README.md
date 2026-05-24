# 0122. Best Time to Buy and Sell Stock II / 买卖股票的最佳时机 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, DP, Array · 贪心, 动态规划, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given prices array (price on day `i`), you may buy/sell multiple times (must sell before next buy, no holding two). Return maximum profit.

**中文**: 给每天的股价, 可以多次买卖 (同一时刻最多持一股), 求最大利润.

## 思路

### Core idea

**累加所有"今天比昨天涨" 的差值** — 仅此一行. 不用真的找 buy/sell pairs.

### Key Insights

1. **数学等价: 区间利润 = 区间内所有日差之和 / Interval profit = sum of daily diffs**

    任何 buy 在 i, sell 在 j 的利润 `prices[j] - prices[i]` 等于:

    ```
    prices[j] - prices[i] = ∑ k∈[i, j-1] (prices[k+1] - prices[k])
    ```

    把所有"先低后高" 的最优 trades 合起来, 等价于**把所有正的日差全收**. 负的日差 (跌日) 跳过 — 不买就是不亏. 累加正差 = 取得最优.

2. **跟 [0121 单次交易] / [0309 含冷冻] / [0714 含手续费] 的关系 / Stock-pricing family**

    | 题 | 限制 | 解法 |
    |---|---|---|
    | [0121](../../10-dp/0121-best-time-to-buy-and-sell-stock/README.md) | 只能买卖 1 次 | 维护 minPrice + 最大 (prices[i] - min), 或状态机 DP |
    | **0122 (本题)** | 不限次数, 无手续费/冷冻 | **贪心: 累加正差** |
    | 0309 (待补) | 不限 + **冷冻 1 天** | DP 状态机 (持 / 不持 / 冷冻) |
    | 0714 (待补) | 不限 + **手续费** | DP 状态机, 卖时扣 fee |
    | 0123 (待补) | **最多 2 次** | DP 五状态 |
    | 0188 (待补) | **最多 k 次** | DP k×2 状态 |

    **加任一额外约束 (冷冻/手续费/次数) 贪心立刻失效, 必须上 DP**. 0122 是这家族里唯一一个可以贪心一行解决的.

3. **DP 解法 (跟 [0121](../../10-dp/0121-best-time-to-buy-and-sell-stock/README.md) 一字之差) / DP — one-line diff from 0121**

    跟 0121 的状态机 DP 同骨架, **只差一行** — 买入那行用 `dp[i-1][0] - prices[i]` 替代 0121 的 `-prices[i]`:

    | 行 | 0121 (一次交易) | **0122 (本题, 无限次)** |
    |---|---|---|
    | 卖出 | `dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])` | **完全一样** |
    | 买入 | `dp[i][1] = max(dp[i-1][1], -prices[i])` | `dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])` |

    **为什么差这一行?** 0121 买入前利润必为 0 (只准一次交易, 买前没卖过); 0122 买入前可以已经买卖很多轮 → 用 `dp[i-1][0]` (昨天不持股的累计利润) 当起点. 把这一行改了, 模板就支持无限次交易.

    > 见下 Solution v2 完整代码. 贪心更短, DP 更通用 — **加任何额外约束 (冷冻/手续费/次数) 都从 DP 版改, 不从贪心版改**.

4. **不要找精确 buy/sell pair / Don't bother identifying trades**

    新手会写"找谷峰 → 累计 peak - valley". 也对, 但代码长. 累加正差是同一答案的最简表达 — 数学等价.

### 一句话总结

**每天涨多少, 全收. `for i: if prices[i] > prices[i-1], res += diff`. 加冷冻/手续费就上 DP, 这题别上.**

### 图解

`prices = [7, 1, 5, 3, 6, 4]`:

```
日差:  -6  +4  -2  +3  -2
收:    -   4   -   3   -
res = 7
```
等价于"day1 买 day2 卖" (+4) + "day3 买 day4 卖" (+3).

## Solution

=== "C++"
    === "v1 贪心: 累加正差 (最短)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int res = 0;
                for (int i = 1; i < (int)prices.size(); i++) {
                    if (prices[i] > prices[i - 1]) res += prices[i] - prices[i - 1];
                }
                return res;
            }
        };
        ```

    === "v2 状态机 DP (跟 0121 一字之差)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int n = prices.size();
                vector<vector<int>> dp(n, vector<int>(2));
                dp[0][0] = 0;
                dp[0][1] = -prices[0];
                for (int i = 1; i < n; i++) {
                    dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);    // 卖
                    dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);    // 买 (跟 0121 的 -prices[i] 不同)
                }
                return dp[n - 1][0];
            }
        };
        ```

    === "v3 状态机 DP + 滚动 O(1)"
        ```cpp
        class Solution {
        public:
            int maxProfit(vector<int>& prices) {
                int hold = -prices[0], cash = 0;
                for (int i = 1; i < (int)prices.size(); i++) {
                    int newCash = max(cash, hold + prices[i]);                 // 卖
                    int newHold = max(hold, cash - prices[i]);                 // 买
                    cash = newCash; hold = newHold;
                }
                return cash;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, prices: list[int]) -> int:
            # v1 贪心: 一行 Pythonic, sum 生成器 + max 隔离正差
            # max(0, x) 把负差替成 0, 等价 C++ if 过滤
            return sum(max(0, prices[i] - prices[i - 1]) for i in range(1, len(prices)))

            # v3 DP 滚动版 (扩展性强, 加约束就从这里改):
            # hold, cash = -prices[0], 0
            # for p in prices[1:]:
            #     cash, hold = max(cash, hold + p), max(hold, cash - p)
            # return cash
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @return {number}
     */
    var maxProfit = function(prices) {
        // v1 贪心
        let res = 0;
        for (let i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i - 1]) res += prices[i] - prices[i - 1];
        }
        return res;
        // v3 DP 滚动: hold = -prices[0], cash = 0; 同 C++ v3
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 易错点

- **加额外约束 (冷冻/手续费/次数限制) 贪心立刻失效**: 0309/0714/0123/0188 全部必须 DP, 别套这题的累加正差. 看到限制条件先想 DP.
- **不要试图找精确 buy/sell pair**: 等价但代码长. 累加正差是同答案的最简形式.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门
- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) — 同款一遍贪心
- [0121. Best Time to Buy and Sell Stock](../../10-dp/0121-best-time-to-buy-and-sell-stock/README.md) — 只能 1 次, 维护 minPrice 或状态机 DP (在 §10)
- 0309\. Best Time to Buy and Sell Stock with Cooldown (待补) — 加 1 天冷冻, DP 三状态
- 0714\. Best Time to Buy and Sell Stock with Transaction Fee (待补) — 加手续费, DP 状态机
- 0123\. Best Time to Buy and Sell Stock III (待补) — 最多 2 次, DP 五状态
- 0188\. Best Time to Buy and Sell Stock IV (待补) — 最多 k 次, DP k×2 状态
