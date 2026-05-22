# 0452. Minimum Number of Arrows to Burst Balloons / 用最少数量的箭引爆气球

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Sort, Interval · 贪心, 排序, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Each balloon is a horizontal interval `[x_start, x_end]`. A vertical arrow shot at x bursts every balloon containing x. Return the minimum number of arrows to burst all balloons.

**中文**: 每个气球是一段水平区间 `[x_start, x_end]`. 在某 x 处垂直射一箭, 会引爆所有覆盖 x 的气球. 求引爆所有气球的最少箭数.

## Key Insights

1. **按右边界升序排 + 贪心射"当前组的最右端" / Sort by right edge, shoot at it**

    Yang 的核心: 按右边界 `points[i][1]` 升序 sort, 第一箭射在第一个气球的右边界 (`arrowPos = points[0][1]`). 这能覆盖**所有左边界 ≤ arrowPos 的气球**.

    > 为什么按右边界而不是左边界? 按右边界排, 第一个气球的右边界是"所有可能与它重叠的气球" 的共同上界. 射在它的右边界, 能贪心地覆盖最多.

2. **判定"需要新箭" 的条件: 当前左边界 > arrowPos / Disjoint = new arrow**

    扫到一个气球, 若它的**左边界** > **上一箭的位置**, 说明跟当前组没有重叠 → 必须新一箭, 位置更新为当前气球的右边界. 否则被旧箭一起打爆, 不动.

    一句话: **左边界擦不上, 就开新箭.**

3. **经典区间贪心模板 / Classic interval scheduling**

    跟 [0435 Non-overlapping Intervals](待补) 是同一个模板 (那题求最少删多少区间使剩下不重叠, 本题求最少箭). 模板:

    ```
    sort by right edge asc;
    last_end = -INF;
    for each interval:
        if interval.left > last_end:   // 不重叠 → 开新组
            count++;
            last_end = interval.right;
        // else 跟上一组合并, last_end 不变 (因为 last_end 已经是组内最小右)
    ```

    "射箭最少 = 最大不重叠区间数 = 必须的分组数". 三种说法同一道题.

4. **溢出小坑: int 比较 / Comparator overflow**

    LC 输入里 `x_start, x_end` 可能接近 INT_MAX. C++ 比较器写 `return a[1] < b[1]` 安全 (没有减法). 别写 `return a[1] - b[1] < 0` — 两个大整数相减会溢出.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findMinArrowShots(vector<vector<int>>& points) {
            if (points.empty()) return 0;
            sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
                return a[1] < b[1];                                // 右边界升序; 不用减法防溢出
            });
            int arrows = 1;
            int arrowPos = points[0][1];                           // 第一箭射在第 1 个气球的右边界
            for (int i = 1; i < (int)points.size(); i++) {
                if (points[i][0] > arrowPos) {                     // 左边界擦不上 → 新箭
                    arrows++;
                    arrowPos = points[i][1];
                }
            }
            return arrows;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findMinArrowShots(self, points: list[list[int]]) -> int:
            if not points:
                return 0
            # key=lambda p: p[1]: 右边界升序排. Python int 任意精度, 没有溢出问题
            points.sort(key=lambda p: p[1])
            arrows = 1
            arrow_pos = points[0][1]
            for p in points[1:]:                                   # 切片跳过第 0 个, Pythonic
                if p[0] > arrow_pos:
                    arrows += 1
                    arrow_pos = p[1]
            return arrows
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} points
     * @return {number}
     */
    var findMinArrowShots = function(points) {
        if (points.length === 0) return 0;
        // 注意: 数字排序的 compareFn (a, b) => a[1] - b[1] 在 INT 接近 2^31 时可能溢出 (LC 测试有大数)
        // 安全写法: 返回 a[1] < b[1] ? -1 : a[1] > b[1] ? 1 : 0
        points.sort((a, b) => a[1] < b[1] ? -1 : a[1] > b[1] ? 1 : 0);
        let arrows = 1;
        let arrowPos = points[0][1];
        for (let i = 1; i < points.length; i++) {
            if (points[i][0] > arrowPos) {
                arrows++;
                arrowPos = points[i][1];
            }
        }
        return arrows;
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导.
- **Space**: O(1) (sort 原地).

## 相关题目

- [0406. Queue Reconstruction by Height](../0406-queue-reconstruction-by-height/README.md) — 同款 sort + 按规则处理
- [0455. Assign Cookies](../0455-assign-cookies/README.md) — sort + 配对贪心
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 同款"按右端排 + 不重叠分组", 直接对照题 (判定差一个等号)
- 0056\. Merge Intervals (待补) — 区间合并, 按**左端**排 + 累积
- 0763\. Partition Labels (待补) — 同款"维护当前组右端" 贪心
- 0253\. Meeting Rooms II (待补) — 区间最少会议室, heap / 差分
