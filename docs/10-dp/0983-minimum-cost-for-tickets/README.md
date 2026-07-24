# 0983. Minimum Cost For Tickets / 最低票价

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Linear · 动态规划, 线性
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-for-tickets/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min cost of 1/7/30-day tickets to cover all travel days** → **linear DP** `dp[i]` = min cost to cover through day i; **travel day** ⇒ `min(cost1 + dp[i-1], cost7 + dp[i-7], cost30 + dp[i-30])`; **non-travel day** ⇒ `dp[i] = dp[i-1]`.
>
> **中文**: **1/7/30 日票覆盖所有出行日最低价** → **线性 DP** `dp[i]` = 到第 i 天最低总花费; **出行日** 取三种票最优, **非出行日** 直接继承 `dp[i-1]`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 一年 365 天. `days` 是**出行日** 列表 (递增, 唯一). `costs[0/1/2]` 分别是 **1/7/30 日票** 的价格. 求**覆盖所有出行日**的最低总花费.

**中文**: 三种票 (1/7/30 天), 求覆盖所有出行日的最低花费.

## Key Insights

1. **🔑 灵魂定义: `dp[i]` = 到第 i 天覆盖所有 (i 及以前的) 出行日的最低花费 / dp[i] = min cost through day i**

    **决策**: **第 i 天是否是出行日?**

    - **不是出行日**: 不需要新票 → `dp[i] = dp[i - 1]`.
    - **是出行日**: 三种票的价格 + 覆盖之前**未包含**日的 dp:
        - 1 日票: `costs[0] + dp[i - 1]`.
        - 7 日票: `costs[1] + dp[i - 7]` (覆盖 [i-6, i]).
        - 30 日票: `costs[2] + dp[i - 30]` (覆盖 [i-29, i]).
    - 取最小值.

    > **"最后一步" 思维**: 到第 i 天, 最后一张票是哪种? 分三种情况求 min. 这是 DP 通用套路 (跟 [DP 通用思维流程](../topic-dp-thinking-process.md) 呼应).

2. **🔑 边界: `max(0, i - k)` 兜负下标 / Boundary: `max(0, i - k)`**

    若 `i < 7`, `i - 7 < 0` → 越界. 用 `max(0, i - 7)` 兜底:

    - `dp[0] = 0` (第 0 天啥都不用买).
    - `dp[max(0, i - 7)]` 若 i < 7 就取 dp[0] = 0. 语义: "7 日票直接覆盖了从第 1 天到今天".

    > **DP 边界的"哨兵"** — `dp[0]` 是 base case, 越界索引都 fallback 到它.

3. **🔑 Yang 的巧妙 marker: `dp[d] = -1` 标出行日 / Clever `-1` marker**

    ```cpp
    vector<int> dp(365 + 1, 0);
    for (int d : days) dp[d] = -1;                          // 出行日标 -1
    for (int i = 1; i <= 365; i++) {
        if (dp[i] != -1) dp[i] = dp[i - 1];                 // 非出行日
        else {
            dp[i] = min({cost1 + dp[i-1], cost7 + dp[max(0, i-7)], cost30 + dp[max(0, i-30)]});
        }
    }
    ```

    - **-1 是占位符** 让 "是不是出行日" 变成 O(1) 判定.
    - **替代方案**: `unordered_set<int> travel(days.begin(), days.end())`, 用 `travel.count(i)` 判. 也 O(1), 更语义化.

    > **"用 int 数组的特殊值当 flag"** 是节省空间的小 hack. 但对可读性稍伤 — set 版更清晰.

4. **🔑 遍历所有 365 天 (不仅 days) / Iterate all 365 days, not just travel days**

    为啥不只对出行日循环?

    - **7 日票和 30 日票** 覆盖了**非出行日** 的区间 — dp 值需要在每一天维护, 才能正确回溯.
    - **`dp[i-1]` 依赖非出行日** 的 dp 值, 所以必须每天都算 (即使只是继承).

    > **"依赖链决定遍历范围"** — 若跳过, `dp[i-7]` 可能没定义.

5. **🔑 返 `dp[days.back()]` — 最后出行日的 dp / Return dp[last travel day]**

    最后出行日之后的 dp[i] 都等于它 (非出行日一路继承). 直接返 `dp[days.back()]` 也可以返 `dp[365]`, 结果相同.

    > **少算一步优化** — 后面继承没意义, 直接停在最后出行日.

6. **🔑 复杂度 O(365) 时间, O(365) 空间 / Constant time (bounded by 365)**

    - **Time**: 365 天 × 3 决策 = O(365) 常量. 对**任何** days 输入都是常量 (因为一年最多 365 天).
    - **Space**: O(365).

    > 少数 LC 题的"复杂度是**真常量**". 数据规模天然有界.

7. **🔑 空间优化 O(30) / Space optimization to O(30)**

    因为 `dp[i]` 只依赖 `dp[i-1], dp[i-7], dp[i-30]` — **只需保留最近 30 天** 的 dp. 用**滚动数组** 降到 O(30). 常数级差别, 不做也无所谓.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int mincostTickets(vector<int>& days, vector<int>& costs) {
            vector<int> dp(365 + 1, 0);
            for (int d : days) dp[d] = -1;                          // 出行日 marker

            for (int i = 1; i <= 365; i++) {
                if (dp[i] != -1) {
                    dp[i] = dp[i - 1];                              // 非出行日, 继承
                } else {
                    dp[i] = min({costs[0] + dp[i - 1],
                                 costs[1] + dp[max(0, i - 7)],
                                 costs[2] + dp[max(0, i - 30)]});    // 三种票取 min
                }
            }
            return dp[days.back()];                                  // 最后出行日就是答案
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def mincostTickets(self, days: list[int], costs: list[int]) -> int:
            # set 版比 marker 更 Pythonic + 可读
            travel = set(days)
            dp = [0] * (366)
            for i in range(1, 366):
                if i not in travel:
                    dp[i] = dp[i - 1]
                else:
                    dp[i] = min(
                        costs[0] + dp[i - 1],
                        costs[1] + dp[max(0, i - 7)],
                        costs[2] + dp[max(0, i - 30)],
                    )
            return dp[days[-1]]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} days
     * @param {number[]} costs
     * @return {number}
     */
    var mincostTickets = function(days, costs) {
        const travel = new Set(days);
        const dp = new Array(366).fill(0);
        for (let i = 1; i <= 365; i++) {
            if (!travel.has(i)) {
                dp[i] = dp[i - 1];
            } else {
                dp[i] = Math.min(
                    costs[0] + dp[i - 1],
                    costs[1] + dp[Math.max(0, i - 7)],
                    costs[2] + dp[Math.max(0, i - 30)]
                );
            }
        }
        return dp[days[days.length - 1]];
    };
    ```

## Complexity

- **Time**: O(365) — 一年天数常量.
- **Space**: O(365) — dp 数组 (可优化到 O(30)).

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP 母题
- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 一维 DP 基础
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 花费型线性 DP
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包 (类似"选票")
- [0139. Word Break](../0139-word-break/README.md) — 线性 DP 判定
- [0887. Super Egg Drop](../0887-super-egg-drop/README.md) — 反向思维 DP
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 序列 DP
- 0790\. Domino and Tromino Tiling (待补) — 线性 DP + 状态机
