# 0057. Insert Interval / 插入区间

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Sweep, Greedy · 数组, 扫描, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/insert-interval/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Intervals are already sorted; a linear 3-phase sweep suffices**: (1) push everything **strictly before** new; (2) **absorb** each overlapping one into `new` by shrinking start / expanding end; (3) push everything **strictly after**. O(n) time, no re-sort.
>
> **中文**: **原数组已排好序, 一遍扫三阶段**: (1) 完全在**新区间左侧**的直接入 res; (2) **重叠**的合并进 new (min start, max end); (3) 完全在**右侧**的直接入 res. O(n), 不用重排.
>
> *Template / 模版*: **Sorted-input sweep with 3 phases** — 当输入已排序, 一次扫比"合并 [0056] 再排一遍" 快一层.

## Problem

**EN**: 给**已排序且无重叠**的区间列表 `intervals` 和新区间 `newInterval`, 插入后合并所有重叠, 返新列表.

**中文**: 有序无重叠区间, 插入新区间并合并.

## Key Insights

1. **🔑 灵魂: 3 阶段扫 O(n) — 不用重排 / 3-phase sweep, no re-sort**

    输入**已按 start 排序且无重叠**, 是宝贵先验. 新区间 `[s, e]` 把原数组切成 3 段:

    ```
    | 完全在左 | 与 new 重叠 | 完全在右 |
    ```

    - **左侧**: `intervals[i][1] < s` — 直接进 res.
    - **重叠**: `intervals[i][0] ≤ e` — 融入 new, 扩其边界.
    - **右侧**: 剩下的直接进 res.

    → **一遍走完**, O(n). 若走"[0056 Merge Intervals](../0056-merge-intervals/README.md) 直接把 new 塞进去再合并" 会**多一次 O(n log n) 排序**, 浪费.

    > **利用已知有序性省一层排序** — 面试若答"排序 + 合并" 会被要求优化到 O(n).

2. **🔑 阶段边界: `< s` vs `≤ s` 判"完全在左" / Strictly-before boundary**

    Yang 用 `intervals[i][1] < newInterval[0]`:

    - **`<` 严格小于**: `[1, 3]` 和 `[3, 5]` **不算左侧** — 因为 3 = 3, 可以合并 (端点相接算重叠).
    - 若用 `≤` 就会误把 `[1, 3]` 归左侧, `[3, 5]` 单独存在, **漏合并**.

    → 端点相接**是否算重叠** 决定 `<` vs `≤`. LC 0057 约定**相接算合并** (合并到一个), 所以 `<`.

    > **易错点 top 1**: 端点相接算不算重叠. 本题算, 故用 `<`.

3. **🔑 阶段边界: `≤ e` 判"重叠" / Overlap boundary**

    Yang: `intervals[i][0] <= newInterval[1]`.

    - `intervals[i]` 已经过了"左侧" 阶段 → `intervals[i][1] ≥ s`.
    - 再判 `intervals[i][0] ≤ e` → 两条件合起来 = **有交集 or 端点相接**.
    - **不用检查 `≤` 还是 `<`**, 因为**合并只要 union 边界**, `<` 或 `≤` 都合. `≤` 更简洁, 少一个 edge case.

4. **🔑 灵魂: 合并 = min start, max end / Merge = union bounds**

    ```cpp
    newInterval[0] = min(intervals[i][0], newInterval[0]);
    newInterval[1] = max(intervals[i][1], newInterval[1]);
    ```

    - **min start**: 新区间起点可能被前面的区间"提前" (例 new=[5,7], 遇到 [3,6] → merged=[3,7]).
    - **max end**: 新区间终点可能被后面的区间"推后" (例 new=[5,7], 遇到 [6,10] → merged=[5,10]).

    → 反复 min/max **就是 union**. 累积所有重叠段的最外边界.

    > **"扩边界" 是区间合并的核心操作**. 见 [0056](../0056-merge-intervals/README.md), [0435](../0435-non-overlapping-intervals/README.md).

5. **🔑 灵魂: `newInterval[0] = min(..., newInterval[0])` 是**幂等的 / Idempotent update**

    因为 `newInterval[0]` **只可能变小或不变**, `min` 是"变小" 语义. **反复调用无副作用**. 若初始 new=[5,7], 依次吃 [3,6], [4,8] → 每次 min(3, 5)=3, min(4, 3)=3 → 稳定收敛到 3.

    → 这种"单调 min/max 累积" 是**扫描 + 状态更新**的通用模式.

6. **🔑 输出 new 一定在第 2 段之后 / Push new after absorb phase**

    Yang 结构:

    ```cpp
    while (阶段1) { res.push_back(intervals[i++]); }
    while (阶段2) { new.absorb(intervals[i++]); }
    res.push_back(newInterval);              // ⚠️ 一定要在 2 之后, 3 之前
    while (阶段3) { res.push_back(intervals[i++]); }
    ```

    - **必须在阶段 2 结束才 push new** — 因为 new 可能还在被合并, 提前 push 会 push 未完成品.
    - 就算**没重叠** (阶段 2 空跑), 也**必须 push** new — 否则漏了新区间.

    > **易错点 top 2**: 阶段 2 若空跑, 别忘还是要 push 一次 new.

7. **🔑 复杂度 / Complexity**

    - **Time**: O(n) — 每个 interval 只被访问 1 次.
    - **Space**: O(n) — 输出数组 (in-place 修改原数组不算).

8. **🔑 变形: 二分找起点加速常数 / Binary search speeds constant, not asymptote**

    若 n 很大且 new 落在末尾, 阶段 1 扫很多. 可用 `upper_bound` **O(log n) 定位** 阶段 1 结束. 但**总仍 O(n)** (输出得复制). 面试非必要, 除非追问"若 new 落末尾如何优化"再上.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
            vector<vector<int>> res;
            int n = intervals.size(), i = 0;

            // 阶段 1: 严格在左 — 直接入 res
            while (i < n && intervals[i][1] < newInterval[0]) {
                res.push_back(intervals[i++]);
            }
            // 阶段 2: 与 new 重叠 — 合并到 new
            while (i < n && intervals[i][0] <= newInterval[1]) {
                newInterval[0] = min(intervals[i][0], newInterval[0]);
                newInterval[1] = max(intervals[i][1], newInterval[1]);
                i++;
            }
            res.push_back(newInterval);                 // ⚠️ 阶段 2 后必推
            // 阶段 3: 严格在右 — 直接入 res
            while (i < n) {
                res.push_back(intervals[i++]);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def insert(self, intervals: list[list[int]], newInterval: list[int]) -> list[list[int]]:
            # Python 无 auto-increment, 用 while + i 手动推进
            # 或用生成器 iter() + next() 更 Pythonic, 但索引写法更直观
            res: list[list[int]] = []
            n, i = len(intervals), 0

            # 阶段 1: 严格在左
            while i < n and intervals[i][1] < newInterval[0]:
                res.append(intervals[i])
                i += 1

            # 阶段 2: 重叠 — 就地扩 new 的边界
            # min/max 天然幂等, 反复调用只会收窄/扩宽单向
            while i < n and intervals[i][0] <= newInterval[1]:
                newInterval[0] = min(newInterval[0], intervals[i][0])
                newInterval[1] = max(newInterval[1], intervals[i][1])
                i += 1
            res.append(newInterval)

            # 阶段 3: 严格在右 — Python 惯用: 直接切片 append
            # extend + 切片比 while 逐个 append 更 Pythonic
            res.extend(intervals[i:])
            return res
    ```

=== "JavaScript"
    ```javascript
    var insert = function(intervals, newInterval) {
        const res = [];
        const n = intervals.length;
        let i = 0;

        // 阶段 1: 严格在左
        while (i < n && intervals[i][1] < newInterval[0]) {
            res.push(intervals[i++]);
        }

        // 阶段 2: 重叠合并
        // Math.min/max 支持任意个参数; 这里两两比较, 展开语法 (...) 不必用
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        res.push(newInterval);

        // 阶段 3: 严格在右
        // 展开语法 ...intervals.slice(i) 更简洁: res.push(...intervals.slice(i));
        // 但 for 循环内存开销更小, 大数据推 for
        while (i < n) {
            res.push(intervals[i++]);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) 输出.

## 相关题目

- [0056. Merge Intervals](../0056-merge-intervals/README.md) — 无有序前提, 需自己排序 + 合并
- [0252. Meeting Rooms](../0252-meeting-rooms/README.md) — 排序 + 相邻不重叠判定
- [0253. Meeting Rooms II](../0253-meeting-rooms-ii/README.md) — 最少几间会议室 (min-heap)
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 排序 + 删除最少个使无重叠
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 区间点覆盖
- 0986\. Interval List Intersections (待补) — 双指针扫两组求交
- 1288\. Remove Covered Intervals (待补) — 排序 + 贪心 remove
- 0759\. Employee Free Time (待补) — 多员工空闲时段, 合并思路
