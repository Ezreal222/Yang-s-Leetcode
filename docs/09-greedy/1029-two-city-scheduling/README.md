# 1029. Two City Scheduling / 两地调度

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Sort · 贪心, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/two-city-scheduling/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `costs[i] = [aCost_i, bCost_i]` — cost to fly person `i` to city A or city B. Exactly `n` people go to A and `n` to B (size `2n`). Return the minimum total cost.

**中文**: `costs[i] = [aCost, bCost]` 是把第 i 人送去 A / B 的费用. 总共 2n 人, 必须刚好 n 人去 A, n 人去 B. 求最小总花费.

## Key Insights

1. **按差价 `aCost - bCost` 升序排 / Sort by diff ascending**

    差价 `aCost - bCost` 是"去 A 比去 B 多花多少":

    - **负值大** (-150): 去 A 比 B **省** 150 → 强烈倾向去 A.
    - **正值大** (+350): 去 A 比 B **贵** 350 → 强烈倾向去 B.

    升序排完, **前 n 个差价最负 → 全去 A**, **后 n 个差价最正 → 全去 B**. 一句话: 让最"想去 A" 的去 A, 最"想去 B" 的去 B.

2. **交换论证 / Exchange argument proves optimality**

    假设最优解里有第 i 人 (sorted 前半) 被分到 B, 必有第 j 人 (后半) 被分到 A. **互换两人** 的成本变化:

    ```
    Δ = (cost[i].a + cost[j].b) - (cost[i].b + cost[j].a)
      = (cost[i].a - cost[i].b) - (cost[j].a - cost[j].b)
      ≤ 0       // 因为 i 在 sorted 前半, 差价更小
    ```

    交换后总成本 ≤ 原来. 重复交换直到完全符合"前半去 A, 后半去 B" — 都不会变差. 局部最优 ⇒ 全局最优.

3. **进阶版 vs Yang 的 v1 / Two equivalent approaches**

    - **v1 (abs sort + counter)**: 按 `|a-b|` 降序排, 优先满足差价最大的偏好, 计数器追踪 A/B 还剩多少名额. 对的, 但要维护两个 counter + 满了切换. 代码长.
    - **v2 (diff sort, 推荐)**: 按 `a-b` 升序排, 前 n 个直接去 A, 后 n 个去 B. **排序结果直接 = 答案的分组**, 没有 counter 也没有 if/else 分支. **少一半代码**, 也更容易证.

    一句话: 排序时**带方向**比"只看绝对偏好" 优雅.

4. **"Sort by diff/cost-difference" 的通用套路 / General pattern**

    任何"两选一 + 名额限制 + 要最小化总花费" 都能套这个模板. 类似的: 0857 Minimum Cost to Hire K Workers (按 wage/quality 排), 1383 Maximum Performance of a Team (类似排序 + heap), 0630 Course Schedule III (按 deadline 排 + heap).

## Solution

=== "C++"
    === "v2 推荐: diff sort"
        ```cpp
        class Solution {
        public:
            int twoCitySchedCost(vector<vector<int>>& costs) {
                // 按 (aCost - bCost) 升序 — 差价最负的排前面
                sort(costs.begin(), costs.end(), [](const vector<int>& a, const vector<int>& b) {
                    return (a[0] - a[1]) < (b[0] - b[1]);
                });
                int n = costs.size() / 2, total = 0;
                for (int i = 0; i < (int)costs.size(); i++) {
                    total += (i < n) ? costs[i][0] : costs[i][1];   // 前半去 A, 后半去 B
                }
                return total;
            }
        };
        ```

    === "v1 (abs sort + counter)"
        ```cpp
        // 思路也对, 但代码长. 留作对照
        class Solution {
        public:
            int twoCitySchedCost(vector<vector<int>>& costs) {
                sort(costs.begin(), costs.end(), [](const vector<int>& a, const vector<int>& b) {
                    return abs(a[0] - a[1]) > abs(b[0] - b[1]);     // 差价 abs 降序
                });
                int a = costs.size() / 2, b = costs.size() / 2, res = 0;
                for (auto& c : costs) {
                    if (a == 0)      { res += c[1]; continue; }
                    if (b == 0)      { res += c[0]; continue; }
                    if (c[0] > c[1]) { res += c[1]; b--; }
                    else             { res += c[0]; a--; }
                }
                return res;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def twoCitySchedCost(self, costs: list[list[int]]) -> int:
            # key=lambda c: c[0] - c[1]: 按差价升序. Python 内建 sort 用 key 比 cmp_to_key 高效
            costs.sort(key=lambda c: c[0] - c[1])
            n = len(costs) // 2
            # 前 n 个去 A (costs[i][0]), 后 n 个去 B (costs[i][1])
            return sum(c[0] for c in costs[:n]) + sum(c[1] for c in costs[n:])
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} costs
     * @return {number}
     */
    var twoCitySchedCost = function(costs) {
        // (a[0] - a[1]) - (b[0] - b[1]) 比较两人的差价大小; 数字范围不大, 不溢出
        costs.sort((a, b) => (a[0] - a[1]) - (b[0] - b[1]));
        const n = costs.length / 2;
        let total = 0;
        for (let i = 0; i < costs.length; i++) {
            total += i < n ? costs[i][0] : costs[i][1];
        }
        return total;
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导.
- **Space**: O(1) (sort 原地).

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同款 sort + 贪心配对入门
- [0406. Queue Reconstruction by Height](../0406-queue-reconstruction-by-height/README.md) — 同款"按规则 sort 直接给出答案" 模板
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) / [0435](../0435-non-overlapping-intervals/README.md) — sort + 区间贪心家族
- 0857\. Minimum Cost to Hire K Workers (待补) — 同款"按比率排" 贪心 + heap
- 0630\. Course Schedule III (待补) — 按 deadline 排 + heap
- 0179\. Largest Number (待补) — 自定义比较器排序 (字符串拼接序)
