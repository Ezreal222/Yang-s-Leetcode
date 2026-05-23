# 1854. Maximum Population Year / 人口最多的年份

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Array, Prefix Sum · 数组, 前缀和 (差分数组)
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-population-year/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `logs[i] = [birth, death]`. A person is alive during years `[birth, death)` (alive in `birth`, NOT alive in `death`). Return the **earliest** year with the maximum population.

**中文**: `logs[i] = [birth, death]`. 一个人活着的年份是 `[birth, death)` (活在 `birth` 那年, 死在 `death` 那年**不算活着**). 求人口最多的最早年份.

## Key Insights

1. **半开区间差分: `[birth, death)` / Half-open interval diff**

    每个人贡献"区间 `[birth, death)` 上 +1". 差分数组写法:

    ```
    diff[birth] += 1;        // 出生年起 +1
    diff[death] -= 1;        // 死亡年起 -1 (那年就不在了)
    ```

    跟 [1094 拼车](../1094-car-pooling/README.md) 完全同款的半开区间语义 — 端点撤销点放在 `death` 而不是 `death + 1`.

2. **年份范围固定 [1950, 2050] → 直接铺数组 / Fixed coord range → array index**

    LC 约束 `1950 <= birth < death <= 2050`. 直接开 `diff[2051]`, 用**年份本身**当下标. 不用离散化, 不用哈希.

    > 一句话: **坐标小 + 整数** → 差分数组完胜 heap / sort. 跟 [1094](../1094-car-pooling/README.md) 同理.

3. **前缀和 + 最大值追踪一遍扫完 / Prefix sum & max in one pass**

    扫年份 1950..2050, 累加 `cur += diff[year]`, 同步追踪 `maxPop` 和 `maxYear`. **从小到大** 扫保证"最早" 自然满足 (相等不更新 → 留下第一个).

    > 关键: `if (cur > maxPop)` 用严格大于. 写成 `>=` 会取最晚年份, 不符合题意.

4. **跟 [1094](../1094-car-pooling/README.md) / [0370](../0370-range-addition/README.md) / [1109](../1109-corporate-flight-bookings/README.md) 的关系 / Diff-array family**

    | 题 | 区间语义 | 求什么 |
    |---|---|---|
    | **1854 (本题)** | 半开 `[birth, death)` | 最大叠加值出现的最早位置 |
    | 1094 拼车 | 半开 `[from, to)` | 任何时刻 ≤ capacity? |
    | 0370 区间加 | 闭 `[start, end]` | 最终每个位置的值 |
    | 1109 航班 | 1-indexed 闭 `[first, last]` | 最终每个位置的值 |

    > 同一模板的不同包装. 区别只在: ① 区间是闭是开; ② 是否 1-indexed; ③ 最后要"全部值" 还是"max/min/某个统计".

5. **不用差分的朴素做法 / Naive baseline**

    每个 log 暴力 `for year in [birth, death): pop[year]++`, 然后扫一遍取 max. 也是 O(n·R) (R = 年份跨度). 这题 R=100 很小, 不会 TLE — 但 LC 写出来仍然不如差分优雅. 一句话: **小规模数据下差分赢的是"代码漂亮", 大规模下赢的是"不 TLE"**.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maximumPopulation(vector<vector<int>>& logs) {
            vector<int> diff(2051, 0);                             // 年份直接当下标; 2050 是上界
            for (auto& log : logs) {
                diff[log[0]] += 1;                                 // birth: +1
                diff[log[1]] -= 1;                                 // death: -1 (半开区间)
            }
            int maxPop = 0, maxYear = 1950, cur = 0;
            for (int year = 1950; year <= 2050; year++) {
                cur += diff[year];                                 // 前缀和 = 当年活着人数
                if (cur > maxPop) {                                // 严格 >, 保证最早年份
                    maxPop  = cur;
                    maxYear = year;
                }
            }
            return maxYear;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maximumPopulation(self, logs: list[list[int]]) -> int:
            # [0] * 2051: 等价 C++ 的 vector<int>(2051, 0)
            diff = [0] * 2051
            for birth, death in logs:                              # 解包二元组
                diff[birth] += 1
                diff[death] -= 1
            # 一遍扫 + 追踪 (max, argmax). 也可以 itertools.accumulate + max + index, 但分两步反而绕
            cur, max_pop, max_year = 0, 0, 1950
            for year in range(1950, 2051):
                cur += diff[year]
                if cur > max_pop:                                  # 严格 > 保证最早年份
                    max_pop, max_year = cur, year
            return max_year
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} logs
     * @return {number}
     */
    var maximumPopulation = function(logs) {
        const diff = new Array(2051).fill(0);
        for (const [birth, death] of logs) {                       // 数组解构
            diff[birth] += 1;
            diff[death] -= 1;
        }
        let cur = 0, maxPop = 0, maxYear = 1950;
        for (let year = 1950; year <= 2050; year++) {
            cur += diff[year];
            if (cur > maxPop) {
                maxPop  = cur;
                maxYear = year;
            }
        }
        return maxYear;
    };
    ```

## Complexity

- **Time**: O(n + R), n = logs, R = 101 (年份跨度).
- **Space**: O(R) — diff 数组.

## 相关题目

- [1094. Car Pooling](../1094-car-pooling/README.md) — 同款半开区间差分
- [0370. Range Addition](../0370-range-addition/README.md) — 闭区间差分模板
- [1109. Corporate Flight Bookings](../1109-corporate-flight-bookings/README.md) — 1-indexed 闭区间差分
- [0253. Meeting Rooms II](../../09-greedy/0253-meeting-rooms-ii/README.md) — 同款"任意时刻最大并发", 坐标无界版 (heap / 扫描线)
- 2848\. Points That Intersect With Cars (待补) — 差分 + 集合大小
- 1893\. Check if All the Integers in a Range Are Covered (待补) — 差分 + 区间覆盖判定
