# 0134. Gas Station / 加油站

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Array · 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/gas-station/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: There are `n` gas stations on a circular route; `gas[i]` is gas at station `i`, `cost[i]` is cost to travel from `i` to `i+1`. Starting with empty tank, find the starting station's index to complete one full circle (clockwise), or `-1` if impossible. Solution is guaranteed unique.

**中文**: 环形路上有 `n` 个加油站, `gas[i]` 是站 i 的油量, `cost[i]` 是从 i 到 i+1 的油耗. 油箱起初为空, 找一个起点使得能跑完一圈. 不存在返回 -1. 答案唯一.

## Key Insights

1. **总油量判定可行性 / `totalTank >= 0` 才有解**

    全程跑下来净收益 = `Σ (gas[i] - cost[i])`. 这个值 ≥ 0 ⇔ 存在合法起点 (否则油不够, 怎么排都跑不完). 一遍扫的时候顺手维护 `totalTank` 就拿到这个判定.

2. **找起点: curTank 负就重置 start = i+1 / Local reset, global validity**

    维护"从当前 start 出发到 i 的累积剩油" `curTank`. 一旦 `curTank < 0`, 说明:

    - 当前 `start` 到 `i` 的任何子起点都凑不出去 — 因为它们的累积比 start 出发更早就开始 (或同时), 必然也是负.
    - 所以新起点必须从 `i + 1` 起重新算 — `start = i + 1; curTank = 0`.

    扫完一遍, 留下的 `start` 就是答案 (前提是 `totalTank >= 0`).

3. **跟 [0053 Maximum Subarray (Kadane)](../0053-maximum-subarray/README.md) 的同源 / Same "negative prefix is poison" 思路**

    Kadane 的"sum < 0 重置 sum" 跟这题"curTank < 0 重置 start" 是**同一个贪心证明** — 负的 prefix 永远是包袱, 丢掉它从下一个位置重起严格更优. 0053 在求最大和, 0134 在求合法起点, 但局部贪心的可证明性完全一致.

4. **O(n) 一遍扫 vs O(n²) 暴力 / Why two counters suffice**

    暴力是每个起点都试一圈, O(n²). 上面的贪心一遍扫, **两个累加器** (curTank 局部 + totalTank 全局) 同时维护就够 — 无需回头, 也无需第二遍验证.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
            int curTank = 0, totalTank = 0, start = 0;
            for (int i = 0; i < (int)gas.size(); i++) {
                int diff = gas[i] - cost[i];
                curTank += diff;
                totalTank += diff;
                if (curTank < 0) {
                    start = i + 1;                       // 当前段不行, 下一站重起
                    curTank = 0;
                }
            }
            return totalTank < 0 ? -1 : start;           // totalTank 决定可行性
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def canCompleteCircuit(self, gas: list[int], cost: list[int]) -> int:
            cur_tank = total_tank = 0
            start = 0
            # zip(gas, cost) 同时迭代两数组, 比双 index 优雅
            for i, (g, c) in enumerate(zip(gas, cost)):
                diff = g - c
                cur_tank += diff
                total_tank += diff
                if cur_tank < 0:
                    start = i + 1
                    cur_tank = 0
            return -1 if total_tank < 0 else start
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} gas
     * @param {number[]} cost
     * @return {number}
     */
    var canCompleteCircuit = function(gas, cost) {
        let curTank = 0, totalTank = 0, start = 0;
        for (let i = 0; i < gas.length; i++) {
            const diff = gas[i] - cost[i];
            curTank += diff;
            totalTank += diff;
            if (curTank < 0) {
                start = i + 1;
                curTank = 0;
            }
        }
        return totalTank < 0 ? -1 : start;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 相关题目

- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) — 同款"负 prefix 重置" 贪心
- [0055. Jump Game](../0055-jump-game/README.md) / [0045. Jump Game II](../0045-jump-game-ii/README.md) — 同款一遍扫维护状态
- [0122. Best Time to Buy and Sell Stock II](../0122-best-time-to-buy-and-sell-stock-ii/README.md) — 同款一遍扫贪心
- 0042\. Trapping Rain Water (待补) — 同款双指针 + 一遍贪心
- 0871\. Minimum Number of Refueling Stops (待补) — 加油站进阶, 贪心 + heap
