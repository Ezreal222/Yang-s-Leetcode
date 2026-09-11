# 0986. Interval List Intersections / 区间列表的交集

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, Array, Interval · 双指针, 数组, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/interval-list-intersections/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Two sorted disjoint interval lists** → merge-like **two-pointer sweep**. At each step, the intersection candidate is `[max(a.lo, b.lo), min(a.hi, b.hi)]`; keep it if `lo ≤ hi`. Then **advance the pointer whose interval ends earlier** (that one is used up for this comparison).
>
> **中文**: **两条有序无重叠区间列表** → 双指针扫. 每步交集 = `[max(左端), min(右端)]`, `lo ≤ hi` 才存. **谁的右端小谁 i++** (它已经贡献完).
>
> *Template / 模版*: **Merge-sort two-pointer with "advance the smaller end"** — 有序双序列求交的通用模板.

## Problem

**EN**: 给两条**已排序且各自无重叠**的闭区间列表 `A`, `B`. 返所有 `A[i] ∩ B[j]` 的非空交集, 按序输出.

**中文**: 两组升序不重叠区间的交集列表.

## Key Insights

1. **🔑 灵魂: 交集公式 `[max(lo), min(hi)]` / Intersection formula**

    任两区间 `[a1, a2]` 和 `[b1, b2]` 的交集 = **`[max(a1, b1), min(a2, b2)]`**.

    - 若 `max(a1, b1) ≤ min(a2, b2)` → 交集非空.
    - 反之无交.

    **直觉**: 交集从**较晚的开始** 开始, 到**较早的结束** 结束.

    > **区间求交** 的经典公式. 面试脱口而出.

2. **🔑 灵魂: 谁的右端小谁 i++ / Advance the smaller-end pointer**

    ```cpp
    if (firstList[i][1] < secondList[j][1]) i++;
    else j++;
    ```

    **推理**: 假设 `A[i].hi < B[j].hi`. `A[i]` 的**右端已经封顶**, 无论 `B[j]` 后面剩多长, `A[i]` 都不能再跟 `B[j]` 的后续 `B[j+1], B[j+2]` 产生更多交集 (因为 `A[i]` 已结束). 但 `B[j]` **还有右侧一截**没被覆盖, 可能跟 `A[i+1]` 有交集. → **应该** `i++`.

    对称地, `B[j].hi < A[i].hi` → `j++`.

    **相等** 时: 两者都结束, 任选一个 `++` 都行. Yang 走 `else j++` 分支.

    > **"advance the smaller end" 是两个有序区间列表求交的通用招**. 也见 [0021 Merge Two Sorted Lists](../../02-linked-list/0021-merge-two-sorted-lists/README.md) 的合并同款思维.

3. **🔑 为什么不是"谁的左端小谁 i++" / Why not advance smaller-start**

    合并两条**有序数字列表**的规则是 "谁小谁 i++". 但**区间不是点** — 区间可以**覆盖**很长一段.

    - 例: `A[i] = [1, 100]`, `B[j] = [2, 3]`. 按"左端小谁 i++" 会 `i++`, 但 `A[i]` 还没用完, 后面 `B[j+1] = [10, 20]` 也跟它有交集. → **左端法漏交集**.
    - 正解: **看谁右端小谁 i++** (谁先结束谁下一位).

    > **易错点 top 1**: 混淆"合并数字" 和"合并区间" 的推进规则.

4. **🔑 灵魂: 为啥能 O(m + n) 走完 / Why linear**

    每次循环**至少一个指针前进**, 最多前进 m + n 次 → **O(m + n)**.

    - 不 backtrack (无重复访问).
    - 每对 `(i, j)` 至多访问 1 次.

    暴力 `for A[i]: for B[j]: intersect` 是 O(m·n). 有序前提让**双指针省一维**.

5. **🔑 输入约定: 闭区间, `lo ≤ hi` / Closed intervals, ≤ check**

    LC 0986 是**闭区间**. 交集判据用 `lo ≤ hi`.

    - `[3, 5] ∩ [5, 7] = [5, 5]` — 单点交集**算**. 用 `≤`.
    - 若是**半开区间**, 用 `<` (单点不算).

    > **易错点 top 2**: 闭 vs 半开. 本题闭 → `≤`.

6. **🔑 边界: 空列表 / Empty inputs**

    - 若 `A` 或 `B` 空, `while` 条件立即 false, 返空. 无需特判.
    - 若都空: 同上.

    → **Yang 代码天然处理**, 不用加 if.

7. **🔑 跟 0057 / 0056 的对比 / vs Insert / Merge Intervals**

    - **0056 Merge Intervals**: 一条列表内部合并.
    - **0057 Insert Interval**: 一条有序 + 一个新区间 → 合并.
    - **0986** (本题): **两条**有序列表求**交集** (非合并).

    → 合并 = **union** (min lo, max hi), 交集 = **intersect** (max lo, min hi). **相反对称**.

    > **min/max 反着写 = union vs intersect 的语义**. 面试举一反三.

8. **🔑 复杂度 / Complexity**

    - **Time**: O(m + n).
    - **Space**: O(1) 辅助 (输出不算).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> intervalIntersection(vector<vector<int>>& firstList,
                                                  vector<vector<int>>& secondList) {
            vector<vector<int>> res;
            int i = 0, j = 0;
            while (i < (int)firstList.size() && j < (int)secondList.size()) {
                int lo = max(firstList[i][0], secondList[j][0]);   // 交集起点
                int hi = min(firstList[i][1], secondList[j][1]);   // 交集终点
                if (lo <= hi) res.push_back({lo, hi});             // 闭区间: ≤

                // 谁右端小谁前进 (它已贡献完)
                if (firstList[i][1] < secondList[j][1]) i++;
                else                                    j++;
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def intervalIntersection(self, A: list[list[int]], B: list[list[int]]) -> list[list[int]]:
            # 双指针经典模板; Python 没 ++, 手动 += 1
            res: list[list[int]] = []
            i = j = 0
            while i < len(A) and j < len(B):
                # min/max 是 Python 内置, 直接用
                # 相当于 C++ 的 max(A[i][0], B[j][0])
                lo = max(A[i][0], B[j][0])
                hi = min(A[i][1], B[j][1])
                if lo <= hi:
                    res.append([lo, hi])

                # 谁右端小谁前进
                if A[i][1] < B[j][1]:
                    i += 1
                else:
                    j += 1
            return res
    ```

=== "JavaScript"
    ```javascript
    var intervalIntersection = function(A, B) {
        // 双指针经典; JS 里 ++ 可用
        const res = [];
        let i = 0, j = 0;
        while (i < A.length && j < B.length) {
            // Math.max / Math.min 是全局函数, 非 Array 方法
            // 相当于 Python 的 max()/min() 内置
            const lo = Math.max(A[i][0], B[j][0]);
            const hi = Math.min(A[i][1], B[j][1]);
            if (lo <= hi) res.push([lo, hi]);

            if (A[i][1] < B[j][1]) i++;
            else                   j++;
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(m + n).
- **Space**: O(1) 辅助.

## 相关题目

- [0056. Merge Intervals](../0056-merge-intervals/README.md) — 单列表内合并 (union)
- [0057. Insert Interval](../0057-insert-interval/README.md) — 有序列表 + 单个新区间 (union)
- [0252. Meeting Rooms](../0252-meeting-rooms/README.md) — 排序 + 相邻不重叠判定
- [0253. Meeting Rooms II](../0253-meeting-rooms-ii/README.md) — 最少会议室数
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 排序 + 贪心保留
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 区间点覆盖
- [0021. Merge Two Sorted Lists](../../02-linked-list/0021-merge-two-sorted-lists/README.md) — 双指针合并母题
- 1288\. Remove Covered Intervals (待补) — 排序 + 贪心 remove
- 0759\. Employee Free Time (待补) — 多员工空闲时段
- 1229\. Meeting Scheduler (待补) — 求满足条件的**首个**交集
