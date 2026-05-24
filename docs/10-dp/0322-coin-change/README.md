# 0322. Coin Change / 零钱兑换

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack · 动态规划, 完全背包
    - **Link**: [LeetCode](https://leetcode.com/problems/coin-change/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给硬币面额 `coins` (每种**无限多枚**) 和金额 `amount`, 返回凑出 `amount` 所需的**最少硬币数**, 凑不出返回 `-1`.

**中文**: 完全背包: 给 `coins` (无限多) 和 `amount`, 求凑出 amount 的最少硬币数, 凑不出返回 `-1`.

## Key Insights

1. **完全背包的 min 变体: `+=` → `min(..., dp[j-coin] + 1)` / Complete knapsack, min flavor**

    跟 [0518 (组合数)](../0518-coin-change-ii/README.md), [0377 (排列数)](../0377-combination-sum-iv/README.md) 同一模板, 只换:

    - `dp[j]` 改"最少硬币数".
    - 转移 `+= dp[j-coin]` → `min(dp[j], dp[j-coin] + 1)`.

    | 完全背包变体 | dp 含义 | 转移 |
    |---|---|---|
    | 0518 组合数 | 方案数 (顺序不计) | `dp[j] += dp[j-coin]` (外 coin) |
    | 0377 排列数 | 序列数 (顺序计) | `dp[j] += dp[j-n]` (外 j) |
    | **0322 最值** | **最少硬币数** | **`dp[j] = min(dp[j], dp[j-coin] + 1)`** |

2. **`dp[0] = 0`: 凑 0 需要 0 枚 / Base case**

    跟 0518/0377 的 `dp[0] = 1` (一种方案) 形成对比 — 最值版本初始为 0 (硬币数), 不是 1.

3. **`INT_MAX` 哨兵 + 加 1 防溢出 / Sentinel + guard against +1 overflow**

    没法凑出的 `j` 用 `INT_MAX` 标记. 转移 `dp[j-coin] + 1` 时, 如果 `dp[j-coin] = INT_MAX` 加 1 就溢出. 用 `if (dp[j-coin] != INT_MAX)` 守一下:

    ```cpp
    if (dp[j - coin] != INT_MAX)
        dp[j] = min(dp[j], dp[j - coin] + 1);
    ```

    替代方案: 用 `amount + 1` 当哨兵 (因为答案最多 amount 枚 1 块硬币), 加 1 不会溢出:

    ```cpp
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;
    ...
    dp[j] = min(dp[j], dp[j - coin] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
    ```

4. **🔑 循环顺序不影响结果 / Loop order doesn't matter — it's min**

    跟 [0518/0377](../0518-coin-change-ii/README.md) 的"组合数 vs 排列数" 不同, 本题是**最值**, 不是计数:

    - 组合数和排列数对"顺序" 敏感 → 循环顺序决定答案.
    - 最少硬币数只关心**用了几枚**, 跟"先用哪枚" 无关 → 怎么遍历都是同一个 min.

    > Yang 写的是"外 coin / 内 j" (0518 风格), 写成"外 j / 内 coin" 也行. 完全等价.

5. **正序 j: 完全背包标配 / Forward j: complete knapsack**

    `dp[j-coin]` 是"已经考虑过这枚 coin" 的状态 → 这枚 coin 被反复用 ⇒ 无限供应. 倒序就退化成 0/1 背包 (每枚最多 1 个).

## Solution

=== "C++"
    === "v1 (Yang 原版): INT_MAX 哨兵"
        ```cpp
        class Solution {
        public:
            int coinChange(vector<int>& coins, int amount) {
                vector<int> dp(amount + 1, INT_MAX);
                dp[0] = 0;
                for (int coin : coins) {
                    for (int j = coin; j <= amount; j++) {         // 正序: 完全背包
                        if (dp[j - coin] != INT_MAX) {             // 防 +1 溢出
                            dp[j] = min(dp[j], dp[j - coin] + 1);
                        }
                    }
                }
                return dp[amount] == INT_MAX ? -1 : dp[amount];
            }
        };
        ```

    === "v2: amount+1 哨兵 (省 if)"
        ```cpp
        class Solution {
        public:
            int coinChange(vector<int>& coins, int amount) {
                vector<int> dp(amount + 1, amount + 1);            // 上界哨兵, +1 不会溢出
                dp[0] = 0;
                for (int coin : coins) {
                    for (int j = coin; j <= amount; j++) {
                        dp[j] = min(dp[j], dp[j - coin] + 1);
                    }
                }
                return dp[amount] > amount ? -1 : dp[amount];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def coinChange(self, coins: list[int], amount: int) -> int:
            # float('inf') 比 INT_MAX 干净: Python int 无上限, inf+1 仍然是 inf, 不溢出
            # 等价 C++ 的 amount+1 哨兵思路
            dp = [float('inf')] * (amount + 1)
            dp[0] = 0
            for coin in coins:
                for j in range(coin, amount + 1):                  # 正序: 完全背包
                    dp[j] = min(dp[j], dp[j - coin] + 1)
            return dp[amount] if dp[amount] != float('inf') else -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} coins
     * @param {number} amount
     * @return {number}
     */
    var coinChange = function(coins, amount) {
        // 用 amount+1 当上界哨兵, 比 Infinity 更省 (Math.min 对 Infinity 也 OK)
        const dp = new Array(amount + 1).fill(amount + 1);
        dp[0] = 0;
        for (const coin of coins) {
            for (let j = coin; j <= amount; j++) {
                dp[j] = Math.min(dp[j], dp[j - coin] + 1);
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    };
    ```

## Complexity

- **Time**: O(n × amount), n = `coins.size()`.
- **Space**: O(amount).

## 相关题目

- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 同 coins, 求**组合数** (`+=`)
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 同模板, 求**排列数** (外 j / 内 nums)
- [0279. Perfect Squares](../0279-perfect-squares/README.md) — 完全背包 min 变体: 平方数当硬币, 求最少个数, 跟本题代码几乎一致
- 0983\. Minimum Cost For Tickets (待补) — 完全背包变体, 按日期分桶
- 0139\. Word Break (待补) — 完全背包判定 (能否拼接出字符串)
