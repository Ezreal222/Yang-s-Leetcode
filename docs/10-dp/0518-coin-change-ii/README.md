# 0518. Coin Change II / 零钱兑换 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack, Counting · 动态规划, 完全背包, 计数
    - **Link**: [LeetCode](https://leetcode.com/problems/coin-change-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给硬币面额数组 `coins` (每种**无限多枚**) 和金额 `amount`, 求**凑出 `amount` 的组合数** (不同顺序算同一种).

**中文**: 给面额数组 `coins` (每种无限多), 求凑出 `amount` 的**组合数** (顺序不计).

## Key Insights

1. **完全背包模板入门: 物品可重复用 → 内层正序 / Complete knapsack: items reusable → forward j**

    完全背包的灵魂跟 [0/1 背包 (0416)](../0416-partition-equal-subset-sum/README.md) 的唯一区别就是**内层 j 的方向**:

    | 题型 | 物品用法 | `j` 遍历 |
    |---|---|---|
    | **0/1 背包** (0416/1049/0494/0474) | 每件 ≤1 次 | **倒序** `j: cap → w` |
    | **完全背包** (本题) | 每件可 ≥0 次 | **正序** `j: w → cap` |

    转移公式**完全一样** — 改个方向就换了语义.

    > **为什么正序 = 完全?** 算 `dp[j]` 用 `dp[j-coin]`, 正序时 `dp[j-coin]` 已经是"考虑过这枚 coin" 的新值 → 这枚 coin 被反复"塞" 多次, 正是"无限供应". 倒序则反之.

2. **状态: dp[j] = 凑出金额 j 的组合数 / dp[j] = ways to reach j**

    转移 = "不用 coin" + "至少用一枚 coin":

    $$dp[j] = \underbrace{dp[j]}_{\text{不用}} + \underbrace{dp[j - \text{coin}]}_{\text{用一枚}}$$

    > 跟 [0494 Target Sum](../0494-target-sum/README.md) 的 `dp[j] += dp[j-n]` 一字不差, 只是这里是完全背包语义.

3. **`dp[0] = 1`: 空集算一种 / Base case — empty selection = 1 way**

    凑出 0 的唯一方法是"什么都不选". 跟所有计数 DP 一样, **空集是有效配置**.

    > 漏写 → 整张表都是 0.

4. **🔑 外层 coin / 内层 amount = 组合数; 反过来 = 排列数 / Loop order determines combinations vs permutations**

    | 顺序 | 题 | 算什么 |
    |---|---|---|
    | **外: coins, 内: j** (本题) | 0518 | **组合数** (顺序不计): `[1,2]` 和 `[2,1]` 同一种 |
    | **外: j, 内: coins** | 0377 Combination Sum IV (待补) | **排列数** (顺序计): `[1,2]` 和 `[2,1]` 算两种 |

    **原理**: 外层枚举 coin 时, 每枚 coin "按顺序" 加入考虑 — 一旦 coin₂ 被加入, coin₁ 已经处理完, 不会再回去和 coin₂ 交换顺序. 所以同一种组合只算一次.

    > 这是完全背包计数最容易翻车的地方. 看到"组合 vs 排列" 就立刻反应"循环顺序".

5. **`unsigned long long` 是过度防御 / int is fine here**

    Yang 用了 `unsigned long long`. LC 518 题目保证答案 ≤ INT_MAX, 中间值也不会超 (每次只累加合法配置). 直接 `int` 就够. 不过保险派也无害.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int change(int amount, vector<int>& coins) {
            vector<int> dp(amount + 1, 0);                         // int 足矣 — 答案保证 ≤ INT_MAX
            dp[0] = 1;                                             // 空集 = 一种
            for (int coin : coins) {                               // 外层: 硬币 → 组合数
                for (int j = coin; j <= amount; j++) {             // 内层 j 正序: 完全背包
                    dp[j] += dp[j - coin];
                }
            }
            return dp[amount];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def change(self, amount: int, coins: list[int]) -> int:
            dp = [0] * (amount + 1)
            dp[0] = 1
            for coin in coins:                                     # 外层 coin → 组合数
                # 正序: 完全背包 (倒序就退化成 0/1 背包)
                for j in range(coin, amount + 1):
                    dp[j] += dp[j - coin]
            return dp[amount]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} amount
     * @param {number[]} coins
     * @return {number}
     */
    var change = function(amount, coins) {
        const dp = new Array(amount + 1).fill(0);
        dp[0] = 1;
        for (const coin of coins) {                                // 外层 coin → 组合
            for (let j = coin; j <= amount; j++) {                 // 正序: 完全背包
                dp[j] += dp[j - coin];
            }
        }
        return dp[amount];
    };
    ```

## Complexity

- **Time**: O(n × amount), n = `coins.size()`.
- **Space**: O(amount).

## 相关题目

- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 0/1 背包母题, 对比"倒序 j"
- [0494. Target Sum](../0494-target-sum/README.md) — 0/1 背包计数版, 跟本题转移式一样但 j 方向相反
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包求**最少硬币数** (`min` 取代 `+=`)
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 完全背包排列数 — 对照本题"外 coin / 内 j" 换成"外 j / 内 coin"
- [0279. Perfect Squares](../0279-perfect-squares/README.md) — 完全背包: 平方数当硬币, 求最少个数
- [0139. Word Break](../0139-word-break/README.md) — 完全背包判定 (字符串拼接)
