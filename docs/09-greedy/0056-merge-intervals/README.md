# 0056. Merge Intervals / 合并区间

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Sort, Interval · 贪心, 排序, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/merge-intervals/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a list of intervals `[start, end]`, merge all overlapping intervals and return the non-overlapping result (sorted).

**中文**: 给一组区间 `[start, end]`, 合并所有重叠的, 返回不重叠区间数组.

## Key Insights

1. **按左边界升序排 + 单次扫合并 / Sort by left edge, single-pass merge**

    sort by `a[0] < b[0]` 之后, 重叠的区间必然**相邻**. 扫一遍, 维护"答案数组的最后一段" 作为当前合并段:

    - `res` 空 **或** `res.back()[1] < interval[0]` (上一段右端 < 当前左端, 不重叠) → 直接 push.
    - 否则重叠 → 把 `res.back()[1]` 更新为 `max(res.back()[1], interval[1])` (合并取右端最大).

    `<` 还是 `<=`? 本题用 `<` (即 `[1,2]` 和 `[2,3]` 算**重叠**, 合并成 `[1,3]`). 想"端点相接不算重叠" 就改 `<=`.

2. **跟 [0763](../0763-partition-labels/README.md) / [0452](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) / [0435](../0435-non-overlapping-intervals/README.md) 是同一族 / Running-right-edge family**

    都是"维护当前组右端 + 扫一遍" 模板. 区别只在三件事:

    | 题 | 按什么排 | 切刀条件 | 段内做什么 |
    |---|---|---|---|
    | **0056 (本题)** | **左端**升序 | `last.right < cur.left` | 取 `max(right)` 合并 |
    | 0763 字母 | 不排 (用 lastPos) | `i == end` | 累加段长 |
    | 0452 气球 | 右端升序 | `cur.left > last.right` | 计数 |
    | 0435 区间 | 右端升序 | `cur.left < last.right` | 删除 |

    > 一句话: **要 merge 按左端排, 要 count 按右端排.** 不同诉求, 不同排序方向.

3. **为什么按左端排就够 / Why sorting by left is enough**

    sort 完后, 若区间 j > i (在数组里更靠后), 则 `j.left ≥ i.left`. 此时:

    - 若 `j.left > i.right` → j 与 i (以及所有更早段) 都不重叠 → 开新段.
    - 否则 `j.left ≤ i.right` → j 与 i 重叠 → 合并, 段右端取 max.

    新合并段的右端只可能"≥ 原来的右端", 所以扫到的下一个 j' 仍然只需和**最新一段**比较, 不需回头看更早. **O(n) 一次扫即可**.

4. **return type 注意 / Return type quirk**

    返回 `vector<vector<int>>` — 是**新数组**, 不是原地改. 想原地省空间也行 (双指针 `write` 覆盖 `read`), 但 LC 测试不在乎, 直接新数组代码清晰.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> merge(vector<vector<int>>& intervals) {
            sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
                return a[0] < b[0];                                // 左端升序
            });
            vector<vector<int>> res;
            for (auto& interval : intervals) {
                if (res.empty() || res.back()[1] < interval[0]) {  // 不重叠 → 新开
                    res.push_back(interval);
                } else {
                    res.back()[1] = max(res.back()[1], interval[1]); // 重叠 → 合并右端
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def merge(self, intervals: list[list[int]]) -> list[list[int]]:
            # key=lambda x: x[0]: 按左端升序. 不写 key 时 list 默认按字典序 (先比 [0] 再比 [1]),
            # 本题等价, 但写 key 意图清楚
            intervals.sort(key=lambda x: x[0])
            res = []
            for s, e in intervals:                                 # 解包: s=start, e=end
                if not res or res[-1][1] < s:
                    res.append([s, e])
                else:
                    res[-1][1] = max(res[-1][1], e)                # 原地改 res 末尾, Python list 可变
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} intervals
     * @return {number[][]}
     */
    var merge = function(intervals) {
        // 数字数组排序必须给 compareFn (字典序会出错: [10,1] 会排在 [2,1] 前)
        intervals.sort((a, b) => a[0] - b[0]);
        const res = [];
        for (const [s, e] of intervals) {                          // 解构: 类似 Python 的 for s, e in
            if (res.length === 0 || res[res.length - 1][1] < s) {
                res.push([s, e]);
            } else {
                res[res.length - 1][1] = Math.max(res[res.length - 1][1], e);
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导.
- **Space**: O(n) 输出 (sort 原地不算).

## 相关题目

- [0763. Partition Labels](../0763-partition-labels/README.md) — 同款"维护当前组右端", 但用 lastPos 不 sort
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 按右端排 + 计数
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 按右端排 + 删除
- 0057\. Insert Interval (待补) — 本题变种, 插入新区间后合并
- 0253\. Meeting Rooms II (待补) — 区间最少会议室, heap / 差分
- 0986\. Interval List Intersections (待补) — 两组有序区间求交, 双指针
