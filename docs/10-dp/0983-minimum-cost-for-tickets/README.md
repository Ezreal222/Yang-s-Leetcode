# 0983. Minimum Cost For Tickets / 最低票价

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Linear, Monotonic Queue · 动态规划, 线性, 单调队列
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-for-tickets/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min cost of 1/7/30-day tickets to cover all travel days** → **v1 linear DP over 365 days**: `dp[i]` = min cost through day i; **v2 iterate only travel days + monotone queue**: track (purchase_day, cost_snapshot) for still-valid 7/30-day tickets, front is cheapest.
>
> **中文**: **1/7/30 日票覆盖所有出行日最低价** → **v1 遍历 365 天线性 DP** (`dp[i]` = 到第 i 天最低); **v2 只遍历出行日 + 单调队列**追踪 7/30 日票有效期, 队首即最省.
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

8. **🔑 v2 灵魂: 只遍历出行日 + 单调队列 / v2 killer: iterate only travel days + monotonic queue**

    v1 的浪费: **遍历 365 天** 但很多天是非出行日的"无脑继承". 若 days 很稀疏 (只 5 个日子), 大部分工作是白做.

    **v2 换思路**: 只遍历 `days`. 对每个出行日 `d`:

    - **两条 queue** 分别追踪"仍有效的 7 日 / 30 日票": 存 `(购买日, 那时的累计 cost)`.
    - 每个新出行日先**弹掉过期票** (`purchase_day + 7 <= d` 就出队).
    - **push 假设今天买 7/30 日票的 cost 快照**: `(d, cost + costs[1/2])`.
    - **今日 cost** = min:
        - `cost + costs[0]` — 今天买 1 日票.
        - `last7.front().second` — 用**最早仍有效的 7 日票** (队首) — 那时的 cost 就是"从当时到现在都被这张票覆盖" 的总成本.
        - `last30.front().second` — 同理 30 日票.

    ```
    队首 = 最早仍有效 = 那时 cost 最小 (因为 cost 单调不减)
    → 直接用 front().second 是最优, 天然的**单调队列**.
    ```

    > **"单调不减 + 时间窗口" → 队首即最优** 是 sliding window / monotonic queue 通用模式. 跟 [0239 Sliding Window Maximum](../../06-stack-queue/0239-sliding-window-maximum/README.md) 同源.

9. **🔑 v1 vs v2 对比 / v1 vs v2**

    | | **v1 遍历 365** | **v2 只遍历 days + queue** |
    |---|---|---|
    | Time | O(365) | O(N) — N = days 数 |
    | Space | O(365) dp | O(N) queue |
    | 关键结构 | dp 数组 | 两条单调队列 |
    | 适用 | 通用 | days 稀疏时更优 |
    | 概念 | DP | DP + monotone queue |

    > **v1 简单直接, v2 优雅高效**. 面试写 v1 稳过, follow-up 掏 v2.

## Solution

=== "C++"

    **v1: 遍历 365 天线性 DP**

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

    **v2: 只遍历 days + 单调队列 (O(N))**

    ```cpp
    class Solution {
    public:
        int mincostTickets(vector<int>& days, vector<int>& costs) {
            queue<pair<int, int>> last7, last30;                    // (购买日, 那时的 cost)
            int cost = 0;
            for (int d : days) {
                // 弹过期票: purchase_day + 7 <= d 就出队
                while (!last7.empty()  && last7.front().first  + 7  <= d) last7.pop();
                while (!last30.empty() && last30.front().first + 30 <= d) last30.pop();
                // 假设今天买 7/30 日票 → 存快照 (购买日, 累计 cost 含此票)
                last7.push({d, cost + costs[1]});
                last30.push({d, cost + costs[2]});
                // 今日 cost = min(1 日票, 队首 7 日票, 队首 30 日票)
                cost = min(cost + costs[0], min(last7.front().second, last30.front().second));
            }
            return cost;
        }
    };
    ```

=== "Python"

    **v2: deque + queue (更简洁)**

    ```python
    from collections import deque

    class Solution:
        def mincostTickets(self, days: list[int], costs: list[int]) -> int:
            # deque 从两端 O(1), 用作 FIFO queue. C++ 用 queue<pair>, Python 直接 deque of tuples
            last7 = deque()   # (购买日, 那时的 cost)
            last30 = deque()
            cost = 0
            for d in days:
                while last7 and last7[0][0] + 7 <= d: last7.popleft()
                while last30 and last30[0][0] + 30 <= d: last30.popleft()
                last7.append((d, cost + costs[1]))
                last30.append((d, cost + costs[2]))
                cost = min(cost + costs[0], last7[0][1], last30[0][1])
            return cost
    ```

=== "JavaScript"

    **v2: 数组当 queue (shift 是 O(N), 但 N 小无所谓)**

    ```javascript
    /**
     * @param {number[]} days
     * @param {number[]} costs
     * @return {number}
     */
    var mincostTickets = function(days, costs) {
        // JS 无原生 deque, 用数组 + shift/push. shift 是 O(N) 但本题 N ≤ 365 常量, 无所谓
        // 若真在意, 可用双端指针模拟
        const last7 = [], last30 = [];
        let cost = 0;
        for (const d of days) {
            while (last7.length  && last7[0][0]  + 7  <= d) last7.shift();
            while (last30.length && last30[0][0] + 30 <= d) last30.shift();
            last7.push([d, cost + costs[1]]);
            last30.push([d, cost + costs[2]]);
            cost = Math.min(cost + costs[0], last7[0][1], last30[0][1]);
        }
        return cost;
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| v1 (365 天 DP) | O(365) | O(365) (可优化 O(30)) |
| **v2 (队列)** | **O(N)** | **O(N)** — N = days 数 |

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP 母题
- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 一维 DP 基础
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 花费型线性 DP
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包 (类似"选票")
- [0139. Word Break](../0139-word-break/README.md) — 线性 DP 判定
- [0887. Super Egg Drop](../0887-super-egg-drop/README.md) — 反向思维 DP
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 序列 DP
- [0239. Sliding Window Maximum](../../06-stack-queue/0239-sliding-window-maximum/README.md) — 单调队列母题 (v2 同源)
- 0790\. Domino and Tromino Tiling (待补) — 线性 DP + 状态机
