# 0188. Best Time to Buy and Sell Stock IV / 买卖股票的最佳时机 IV

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, State Machine, Array · 动态规划, 状态机, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给每日股价 `prices` 和 `k`. 最多做 `k` 次 buy+sell, 求最大利润.

**中文**: 最多 k 次买卖, 求最大利润.

## Key Insights

1. **🔑 [0123](../0123-best-time-to-buy-and-sell-stock-iii/README.md) 的泛化: 把 4 状态扩成 `2k` / Generalization of 0123: 4 → 2k states**

    0123 把"第 1 次买/卖, 第 2 次买/卖" 当 4 个状态. 本题改成"第 1..k 次", 状态数变 `2k`. 转移**一字不差** — 第 j 次买依赖第 (j-1) 次卖, 第 j 次卖依赖第 j 次买.

    | 题 | k | 状态数 (无 state 0) | 滚动写法 |
    |---|---|---|---|
    | [0121](../0121-best-time-to-buy-and-sell-stock/README.md) | 1 | 2 (1 buy + 1 sell) | `hold, cash` |
    | [0123](../0123-best-time-to-buy-and-sell-stock-iii/README.md) | 2 | 4 (2 buy + 2 sell) | `buy1, sell1, buy2, sell2` |
    | **0188 (本题)** | k | **2k** | 2k 长数组 |

    > 一旦 k 变量, 4 个 if 就写不动了, 必须用 loop. 这是从 0123 升级到 0188 的唯一难点.

2. **状态编码: `2j = 第 j 次买入后`, `2j+1 = 第 j 次卖出后` (0-indexed) / Index encoding**

    Yang v1 用一维 `dp[2k]`:

    | 索引 | 含义 |
    |---|---|
    | `dp[0]` | 第 0 次买入后 (持有第 1 股) |
    | `dp[1]` | 第 0 次卖出后 (完成 1 次交易) |
    | `dp[2]` | 第 1 次买入后 (持有第 2 股) |
    | `dp[3]` | 第 1 次卖出后 (完成 2 次交易) |
    | … | … |
    | `dp[2k-1]` | 第 (k-1) 次卖出后 (完成 k 次交易, **答案**) |

    > 偶数索引 = 买后, 奇数索引 = 卖后. **奇偶分清** 是这题最容易翻车的地方.

3. **🔑 第 0 次买的"前驱卖" 要特判 / `j == 0` boundary**

    转移: `dp[2j] = max(dp[2j], prevSell - prices[i])` 里的 `prevSell`:

    - `j > 0`: `prevSell = dp[2j - 1]` (上一次卖出后的状态)
    - **`j == 0`: `prevSell = 0`** (从没操作过, 利润 0)

    Yang 用三元 `(j == 0) ? 0 : dp[2*j - 1]` 干掉了越界. 写漏直接 RE 或 WA.

    > 等价的 v2 写法用 `2k+1` 个状态显式包含 `dp[0] = 0` ("没操作"), 直接 `dp[i][j] = max(dp[i-1][j], dp[i-1][j-1] ± prices[i])`, **不需要 if 特判**. 二维版更整齐但多用一个状态.

4. **顺序级联 (跟 0123 同套路) / Sequential cascade carries over from 0123**

    v1 一维滚动: `j` 从小到大遍历, 第 `j` 次买立刻用同次买结果算同次卖. 安全性见 [0123 Key Insight #5](../0123-best-time-to-buy-and-sell-stock-iii/README.md) — 同日 buy+sell 利润 0, max 不会被更新.

5. **优化: `k >= n/2` 时退化成无限次 / Degenerate to 0122 when k is huge**

    一次完整交易至少占 2 天 (1 买 1 卖), 所以 n 天最多 `n/2` 次交易. 如果 `k >= n/2`, 完全等价 [0122](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) (无限次 + 贪心累加正差), O(n).

    ```cpp
    if (k >= n / 2) return greedy(prices);
    ```

    不加这条也对, 只是数组开大浪费. LC 测试里 k 可达 100, n 也可达 1000, 不加也能过.

6. **初始化: 所有"买入" 状态初值 `-prices[0]` / Init all buy states to -prices[0]**

    第 0 天结束时, 任何"第 j 次买入后" 的状态都相当于"反复同天买卖再买", 净支出就是 `-prices[0]`. 所有"卖出后" 状态初值是 0 (默认 vector 初值).

## Solution

=== "C++"
    === "v1 推荐: 一维滚动 O(k)"
        ```cpp
        class Solution {
        public:
            int maxProfit(int k, vector<int>& prices) {
                int n = prices.size();
                // 2j = 第 j 次买入后, 2j+1 = 第 j 次卖出后 (0-indexed)
                vector<int> dp(2 * k, 0);
                for (int j = 0; j < k; j++) dp[2 * j] = -prices[0];    // 所有买入初始
                for (int i = 1; i < n; i++) {
                    for (int j = 0; j < k; j++) {
                        // 第 0 次买没"前驱卖", 用 0; 第 j>0 次买的前驱是 dp[2j-1]
                        int prevSell = (j == 0) ? 0 : dp[2 * j - 1];
                        dp[2 * j]     = max(dp[2 * j],     prevSell - prices[i]);    // 买
                        dp[2 * j + 1] = max(dp[2 * j + 1], dp[2 * j] + prices[i]);   // 卖 (用刚更新的买)
                    }
                }
                return dp[2 * k - 1];
            }
        };
        ```

    === "v2: 二维 2k+1 状态 (含 state 0, 无特判)"
        ```cpp
        class Solution {
        public:
            int maxProfit(int k, vector<int>& prices) {
                int n = prices.size();
                if (n == 0 || k == 0) return 0;
                // dp[i][j]: 第 i 天结束, 状态 j 的最大利润
                // j=0: 未操作; 奇数 j: 第 (j+1)/2 次买后持有; 偶数 j>0: 第 j/2 次卖后
                vector<vector<int>> dp(n, vector<int>(2 * k + 1, 0));
                for (int j = 1; j <= 2 * k; j += 2) dp[0][j] = -prices[0];    // 所有买入初始
                for (int i = 1; i < n; i++) {
                    for (int j = 1; j <= 2 * k; j++) {
                        if (j % 2 == 1) {                                          // 奇: 买
                            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - 1] - prices[i]);
                        } else {                                                    // 偶: 卖
                            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - 1] + prices[i]);
                        }
                    }
                }
                return dp[n - 1][2 * k];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def maxProfit(self, k: int, prices: list[int]) -> int:
            n = len(prices)
            if n == 0 or k == 0:
                return 0

            # k >= n/2 时退化成无限次 (0122 贪心), 提速
            if k >= n // 2:
                return sum(max(0, prices[i] - prices[i - 1]) for i in range(1, n))

            # 一维滚动, 跟 C++ v1 一致
            # 2j = 第 j 次买后, 2j+1 = 第 j 次卖后
            dp = [0] * (2 * k)
            for j in range(k):
                dp[2 * j] = -prices[0]
            for p in prices[1:]:
                for j in range(k):
                    prev_sell = 0 if j == 0 else dp[2 * j - 1]         # 第 0 次买无前驱
                    dp[2 * j]     = max(dp[2 * j],     prev_sell - p)
                    dp[2 * j + 1] = max(dp[2 * j + 1], dp[2 * j] + p)
            return dp[2 * k - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} k
     * @param {number[]} prices
     * @return {number}
     */
    var maxProfit = function(k, prices) {
        const n = prices.length;
        if (n === 0 || k === 0) return 0;
        const dp = new Array(2 * k).fill(0);
        for (let j = 0; j < k; j++) dp[2 * j] = -prices[0];
        for (let i = 1; i < n; i++) {
            for (let j = 0; j < k; j++) {
                const prevSell = j === 0 ? 0 : dp[2 * j - 1];
                dp[2 * j]     = Math.max(dp[2 * j],     prevSell - prices[i]);
                dp[2 * j + 1] = Math.max(dp[2 * j + 1], dp[2 * j] + prices[i]);
            }
        }
        return dp[2 * k - 1];
    };
    ```

## Complexity

- **Time**: O(n × k).
- **Space**: O(k) (v1 一维) / O(n × k) (v2 二维).

## 相关题目

- [0121. Best Time to Buy and Sell Stock](../0121-best-time-to-buy-and-sell-stock/README.md) — k=1 特例
- [0122. Best Time to Buy and Sell Stock II](../../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md) — `k=∞` 特例 (贪心)
- [0123. Best Time to Buy and Sell Stock III](../0123-best-time-to-buy-and-sell-stock-iii/README.md) — k=2 特例
- 0309\. Best Time to Buy and Sell Stock with Cooldown (待补) — k=∞ + 冷冻期
- 0714\. Best Time to Buy and Sell Stock with Transaction Fee (待补) — k=∞ + 手续费
- [§10 DP 思维流程 — 状态机 DP](../topic-dp-thinking-process.md) — 多状态 DP 模板的极致
