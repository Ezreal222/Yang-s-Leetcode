# 0252. Meeting Rooms / 会议室

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Sorting, Greedy, Array · 排序, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/meeting-rooms/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Can one person attend all meetings?** → **sort by start time, check adjacent no-overlap**. If any `intervals[i].start < intervals[i-1].end`, they collide → return false.
>
> **中文**: **一个人能否参加所有会议** → **按开始时间排序, 相邻不重叠**. 若 `start[i] < end[i-1]` 就冲突.
>
> *Template / 模版*: **Sort by start → sweep adjacent pairs** — 区间冲突判定的最简模式.

## Problem

**EN**: 给会议时段 `[start, end)` (半开), 判定单人能否全参加 (无重叠).

**中文**: 无冲突则 true.

## Key Insights

1. **🔑 灵魂: 排序把 O(n²) 两两比较降到 O(n log n) / Sorting reduces pairwise to adjacent**

    暴力: 每对 `(i, j)` 判是否重叠 → **O(n²)**.

    排序后: 若**排序后相邻两个不冲突**, 那**任意两个都不冲突** — 因为后来的 start ≥ 前一个的 start ≥ 前面所有 end 的下界. → **只查相邻 n-1 对**.

    > **"排序后只需扫一遍" 是区间/贪心的黄金法则**. 也见 [0056 Merge Intervals](../0056-merge-intervals/README.md), [0435 Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md).

2. **🔑 灵魂: 半开区间 `[start, end)` 决定判据 `<` / Half-open ⇒ strict less than**

    LC 会议室约定**半开** — 会议 `[10, 20)` 和 `[20, 30)` **不冲突** (10 点开始, 20 点结束 = 20 点前已散场, 20 点新会议正好开始).

    → 冲突判据: **`intervals[i][0] < intervals[i-1][1]`** — 严格小于 `<`, 而非 `≤`.

    若题目是**闭区间**, 用 `≤`. **看清约定**.

    > **易错点 top 1**: `<` vs `≤` 一字之差, 半开/闭区间反着写 → WA.

3. **🔑 排序按 start 天然是二维 lexicographic / vector<vector<int>> 默认按字典序**

    `sort(intervals.begin(), intervals.end())` **默认按 `vector<int>` 的字典序** — 先比 `[0]` (start), start 相同才比 `[1]` (end).

    → 我们只关心 start, 但字典序自动 tie-break by end, **无害且免费**.

    > **C++ 里 `vector<vector<int>>` 默认字典序排** — 面试常用招, 不用写 comparator.

4. **🔑 跟 0253 的关系: "能否" vs "需要几间" / 0252 vs 0253**

    - **0252** (本题): 一个人参加全部会议? → **相邻不重叠即可**. 简单.
    - **0253 Meeting Rooms II** (§09 里已有): 最少需**几间**会议室能装下所有? → **min-heap 扫时间线** 或 **差分数组**.

    → 0252 = 0253 的**判定版**. "0253 需 ≤ 1 间房" 等价于本题 true.

    > 面试若 0252 秒答, 追问会往 0253 走. 提前熟.

5. **🔑 O(n²) 暴力 (小 n 也 OK) / Brute force**

    ```cpp
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (max(start[i], start[j]) < min(end[i], end[j])) return false;
    return true;
    ```

    - `max(start) < min(end)` = 两区间有交集.
    - **n ≤ 10⁴** 时 O(n²) = 10⁸, 卡时间. 排序法**永远推**.

6. **🔑 复杂度 / Complexity**

    - **Time**: O(n log n) — 排序主导. 扫描 O(n).
    - **Space**: O(1) (排序 in-place) 或 O(log n) 栈 (若用 introsort).

7. **🔑 变形: 返冲突对数 / Variant: count conflicts**

    若题目改成"统计冲突次数", 排序后扫相邻不够 (相邻不冲不代表**别对**不冲, 因为**每对**都要判 — 但只有**相邻**能冲, 排序性质保证). 所以还是 O(n).

    → 相邻不冲 ⇒ 传递到全体, 这个**归纳不变量**是排序解的灵魂.

8. **🔑 易错点 top 2 / Pitfalls**

    - **半开 vs 闭区间**: 用 `<` 还是 `≤`. 本题半开 → `<`.
    - **不排序直接扫**: 输入未必按时间给, 必须先 sort.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool canAttendMeetings(vector<vector<int>>& intervals) {
            sort(intervals.begin(), intervals.end());       // 按 start 字典序
            for (int i = 1; i < (int)intervals.size(); i++) {
                if (intervals[i][0] < intervals[i - 1][1])  // 半开: 严格 <
                    return false;
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def canAttendMeetings(self, intervals: list[list[int]]) -> bool:
            # sort() 按元素默认排序; list of list 走字典序, 跟 C++ 一样
            # key=lambda x: x[0] 显式版, 不写也 OK
            intervals.sort()

            # zip(a, a[1:]) 遍历相邻对, Python 惯用招 (无索引变量)
            # 相当于 C++ 的 for i in [1..n) 里的 (intervals[i-1], intervals[i])
            for prev, curr in zip(intervals, intervals[1:]):
                if curr[0] < prev[1]:                       # 半开: 严格 <
                    return False
            return True
    ```

=== "JavaScript"
    ```javascript
    var canAttendMeetings = function(intervals) {
        // 显式 comparator: JS 默认 sort 按字符串 → 数字 sort 必须给 (a, b) => a - b
        // 二维数组按 [0] 排: (a, b) => a[0] - b[0]
        // 若 [0] 相同再比 [1]: (a, b) => a[0] - b[0] || a[1] - b[1] (本题无所谓)
        intervals.sort((a, b) => a[0] - b[0]);

        for (let i = 1; i < intervals.length; i++) {
            if (intervals[i][0] < intervals[i - 1][1]) return false;  // 半开: 严格 <
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n log n).
- **Space**: O(1) 或 O(log n).

## 相关题目

- [0253. Meeting Rooms II](../0253-meeting-rooms-ii/README.md) — 姐妹题, "需几间房" 版, min-heap 或差分
- [0056. Merge Intervals](../0056-merge-intervals/README.md) — 排序 + 相邻合并
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 排序 + 贪心保留
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 区间点覆盖
- 0056\. Merge Intervals — 已上
- 0057\. Insert Interval (待补) — 排序基础上再插一个
- 0986\. Interval List Intersections (待补) — 双指针扫两组
- 1288\. Remove Covered Intervals (待补) — 排序 + 贪心
- 0759\. Employee Free Time (待补) — 多员工空闲时段, 扫时间线
