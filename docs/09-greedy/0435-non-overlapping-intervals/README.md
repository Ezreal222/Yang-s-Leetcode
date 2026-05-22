# 0435. Non-overlapping Intervals / 无重叠区间

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Sort, Interval · 贪心, 排序, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/non-overlapping-intervals/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a list of intervals, return the **minimum number** to remove so the rest are non-overlapping (intervals only touching at endpoints, like `[1,2]` and `[2,3]`, do **not** count as overlapping).

**中文**: 给一组区间, 求**最少删多少**使剩下不重叠 (只在端点相碰, 例如 `[1,2]` 和 `[2,3]`, **不算**重叠).

## Key Insights

1. **按右边界升序排 + 贪心保留"最早结束" / Sort by right edge, keep earliest end**

    Yang 的核心: 按 `intervals[i][1]` 升序 sort. 贪心保留每一组**右边界最小**的区间 — 给后面留出最多空间, 后续能塞进更多不重叠区间.

    扫的时候: `prevEnd` = 上一个保留区间的右边界. 当前区间若不重叠就更新 `prevEnd`; 重叠就**删它** (`count++`, prevEnd 不动 — 因为我们保留更早结束的那个).

2. **重叠判定: `<` 不是 `<=` / Touching endpoints don't overlap**

    `intervals[i][0] < prevEnd` 才算重叠. 等号不算 — 题目明确 `[1,2]` 跟 `[2,3]` 是"端点相碰", 不视为重叠.

    跟 [0452 Arrows](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) 的关键差异: 那题用 `>` (即 `<=` 算重叠/可一箭射爆). **同模板, 看题目对"端点相碰" 的定义切判定符**.

3. **跟 [0452] 是同一题的对偶 / Dual of arrows problem**

    | 题 | 求什么 | 判定 |
    |---|---|---|
    | [0452](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) | 最少箭数 = 最大不重叠分组数 | `>` (擦上算覆盖) |
    | **0435 (本题)** | 最少删数 = n − 最大不重叠数 | `<` (擦上不算重叠) |

    一句话: **`n − 0435 答案 = 0452 答案 (如果用相同的"端点相碰" 定义)`**. 两题骨架一字不差.

4. **为什么按右边界排 (不按左边界) / Right-edge sort is the load-bearing choice**

    经典区间调度结论: 求"最大不重叠区间数" 的最优策略 = 每次贪心选当前**结束最早**的区间. 按右边界 sort 后, 第一个就是结束最早的. 按左边界排不能直接给"结束最早", 还要额外维护.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int eraseOverlapIntervals(vector<vector<int>>& intervals) {
            if (intervals.empty()) return 0;
            sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
                return a[1] < b[1];                              // 右边界升序; 不用减法防溢出
            });
            int count = 0;
            int prevEnd = intervals[0][1];                       // 第一个总是保留
            for (int i = 1; i < (int)intervals.size(); i++) {
                if (intervals[i][0] < prevEnd) {                 // 严格 < 才重叠 (端点碰不算)
                    count++;                                     // 删当前 (保留早结束的)
                } else {
                    prevEnd = intervals[i][1];                   // 保留, 更新边界
                }
            }
            return count;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def eraseOverlapIntervals(self, intervals: list[list[int]]) -> int:
            if not intervals:
                return 0
            intervals.sort(key=lambda x: x[1])                   # 右边界升序
            count = 0
            prev_end = intervals[0][1]
            for s, e in intervals[1:]:                           # tuple unpack 直接拿 (start, end)
                if s < prev_end:
                    count += 1                                   # 重叠, 删当前
                else:
                    prev_end = e                                 # 不重叠, 保留
            return count
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} intervals
     * @return {number}
     */
    var eraseOverlapIntervals = function(intervals) {
        if (intervals.length === 0) return 0;
        // 防溢出: 用三元 < / > 比较, 不用 a[1] - b[1]
        intervals.sort((a, b) => a[1] < b[1] ? -1 : a[1] > b[1] ? 1 : 0);
        let count = 0;
        let prevEnd = intervals[0][1];
        for (let i = 1; i < intervals.length; i++) {
            if (intervals[i][0] < prevEnd) {
                count++;
            } else {
                prevEnd = intervals[i][1];
            }
        }
        return count;
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导.
- **Space**: O(1) (sort 原地).

## 相关题目

- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 直接对偶题, 同模板, 判定差一个等号
- [0406. Queue Reconstruction by Height](../0406-queue-reconstruction-by-height/README.md) — 同款 sort-first 贪心
- 0056\. Merge Intervals (待补) — 区间合并, 按**左边界**排
- 0763\. Partition Labels (待补) — 同款"维护当前组右端" 贪心
- 0253\. Meeting Rooms II (待补) — 区间最少会议室, heap / 差分
- 0646\. Maximum Length of Pair Chain (待补) — 等价"最大不重叠数" = `n - 0435`
