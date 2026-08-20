# 0973. K Closest Points to Origin / 最接近原点的 K 个点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Heap, Priority Queue, Quickselect, Top-K · 堆, 优先队列, 快速选择, Top-K
    - **Link**: [LeetCode](https://leetcode.com/problems/k-closest-points-to-origin/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **K closest points to (0,0)** → **max-heap of size k** on squared-distance: push each, pop when size > k (kicks farthest). Compare on `x² + y²` — no `sqrt` needed.
>
> **中文**: **距离原点最近的 K 个点** → **大小 k 的最大堆** on 距离平方: 每个 push, 超 k 就 pop (踢最远). 用 `x²+y²` 比较, 不用开根.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给点数组 `points`, 每个点 (x, y). 返回**距离原点最近的 K 个点** (顺序任意). 欧氏距离.

**中文**: 找距原点最近的 K 个点.

## Key Insights

1. **🔑 Top-K 模板: max-heap of size k / Top-K template**

    要**最近的 k 个** (即距离最小的 k 个) → **维护 k 大小的 max-heap** on distance:

    - **push** 每个点.
    - **size > k → pop** (**踢最远的**, 因为 max-heap 顶就是最远).
    - 遍历完, 堆里剩的**就是** k 个最近的.

    > **通用模板**: **求 k 个最小 → max-heap of k**, **求 k 个最大 → min-heap of k**. 记牢方向.

2. **🔑 灵魂优化: `x² + y²` 不开根 / Compare on squared distance**

    比较距离大小**不需要开方**:

    ```
    √(x1² + y1²) < √(x2² + y2²)  ⇔  x1² + y1² < x2² + y2²
    ```

    → 直接**存 x² + y²** — 省一次 sqrt 调用, 也**避免浮点误差**.

    > **"求最值不需精确, 只需**保序**变换"** — 一个通用优化思想. `x² < y²` 保序等价 `x < y` (非负数下).

3. **🔑 存 `(distance², index)` — 方便回填结果 / Store index, look up later**

    Yang 的巧: heap 里存 **`pair<int, int>` = (distance², index)**:

    ```cpp
    priority_queue<pair<int, int>> pq;      // 默认 max-heap on .first
    pq.push({d, i});                        // d = x²+y², i = 原下标
    ...
    while (!pq.empty()) {
        auto [_, i] = pq.top(); pq.pop();
        res.push_back(points[i]);            // 用 index 回填
    }
    ```

    - **存索引** 而非点本身: 点是 `vector<int>` 挺大, 存 int 更省 (2×8 vs 24+ bytes).
    - **回填**: 结束遍历堆, 用 index 从 `points` 拿原坐标.

    > **"heap 只存必要信息 (index or key)"** 是内存优化通用技巧. 大对象搜索 Top-K 都这么做.

4. **🔑 `priority_queue<pair<int, int>>` 默认按 first / Default pair comparison**

    C++ `pair` 默认按 **first 升序 → second 升序**. `priority_queue<pair>` 默认 max-heap → 按 **first 降序** (最大在顶).

    → **top() = 最大 distance²** — 正是我们要**踢**的对象.

    > **`pair<distance, id>` 是 Top-K 的常用模式**. distance 决定优先级, id 拿数据. 跟 [0347](../0347-top-k-frequent-elements/README.md) 的 `pair<count, value>` 同源.

5. **🔑 备选招: quickselect O(n) / Alternative: quickselect**

    跟 [0215 Kth Largest](../0215-kth-largest-element-in-an-array/README.md) v2 一样, quickselect 平均 O(n):

    - 用 `nth_element(begin, begin+k, end, cmp)` 一行调用 STL.
    - 或手写 partition + 递归.

    ```cpp
    nth_element(points.begin(), points.begin() + k, points.end(),
                [](auto& a, auto& b) {
                    return a[0]*a[0]+a[1]*a[1] < b[0]*b[0]+b[1]*b[1];
                });
    return vector<vector<int>>(points.begin(), points.begin() + k);
    ```

    > **面试 follow-up "还能更快?"** → 掏 quickselect. LC constraints 通常 heap 够快.

6. **🔑 三种方法对比 / Three approaches**

    | 方法 | Time | Space | 备注 |
    |---|---|---|---|
    | 排序 + 前 k 个 | O(n log n) | O(1) | 最简, 慢 |
    | **max-heap of k** (Yang) | **O(n log k)** | **O(k)** | k ≪ n 时最省 |
    | Quickselect / `nth_element` | O(n) 期望 | O(1) 原地 | 最快但不稳 |

    > **三种都会** = 面试灵活. 从简单到炫: sort → heap → quickselect.

7. **🔑 复杂度 O(n log k) 时间, O(k) 空间 / Complexity**

    - Time: n 个点各 push+可能 pop, 每次 O(log k).
    - Space: 堆容量 k.

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to return the **K points closest to the origin (0, 0)** from a list of 2D points. Let me confirm:"

- "**Euclidean distance**, right? Not Manhattan?" *(yes, Euclidean.)*
- "**Order of the output** matters? Can I return the K points in any order?" *(any order — that unlocks options.)*
- "Are the points **guaranteed distinct**? And is K ≤ len(points)?" *(usually yes on LC — but I'll code without assuming distinctness.)*
- "**Coordinate range**? Just to think about overflow." *(if x, y ≤ 10⁴, then x²+y² fits in int; larger might need long.)*

### 2. Brainstorm approaches (1 min)

> "Three natural approaches, from simplest to fanciest.
>
> **Approach 1 — sort by distance, take first K**. O(n log n). Simple, correct, but sorts more than needed.
>
> **Approach 2 — max-heap of size K**. Iterate through points, push each into a max-heap, and if size > K pop the top (which is the farthest so far). At the end the heap has the K closest. O(n log K), O(K) space. Great when K ≪ n.
>
> **Approach 3 — quickselect** (or `std::nth_element`). Partition the array around a pivot such that the first K elements are the K smallest by distance. **O(n) expected**, O(1) extra. Fastest asymptotically, but has O(n²) worst case if pivots are bad — needs random pivot.
>
> I'll go with **approach 2 (heap)** — it's the most robust and clearly O(n log K). If they ask for faster I'll show quickselect."

### 3. Sketch the algorithm before coding (1 min)

> "Key design decisions I'll call out before coding:
>
> 1. **Squared distance, not sqrt**. Comparing `x²+y²` gives the same ordering as `√(x²+y²)` for non-negative numbers — so I skip the `sqrt`, avoiding floating point and saving cycles.
> 2. **Heap stores `(dist², index)` not the point itself**. Points are `vector<int>` — copying them into the heap is wasteful. Store just an int index and look up `points[i]` at the end.
> 3. **`priority_queue<pair<int, int>>` in C++ is a max-heap on `.first`** by default. That's exactly what I want — the top is the farthest of the current K candidates, so `pop()` kicks the right element.
>
> The loop is 3 lines: compute `d`, push `(d, i)`, if `size > k` pop."

### 4. Code while narrating (2 min)

```cpp
vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
    priority_queue<pair<int, int>> pq;    // max-heap on distance²
    for (int i = 0; i < (int)points.size(); i++) {
        int d = points[i][0] * points[i][0]
              + points[i][1] * points[i][1];
        pq.push({d, i});
        if ((int)pq.size() > k) pq.pop();  // kick the farthest
    }
    vector<vector<int>> res;
    while (!pq.empty()) {
        auto [_, i] = pq.top(); pq.pop();
        res.push_back(points[i]);
    }
    return res;
}
```

> "About 10 lines. The heap does the sorting for me, and I only ever hold K items — that's the win over full-sort."

### 5. Trace an example (1 min)

> "Let me sanity-check with `points = [[1,3],[-2,2],[5,8],[0,1]]`, `k = 2`:
>
> - i=0: d=10, push (10,0). heap=[(10,0)]. size 1 ≤ 2.
> - i=1: d=8, push (8,1). heap=[(10,0),(8,1)]. size 2 ≤ 2.
> - i=2: d=89, push (89,2). heap top=(89,2). size 3 > 2 → pop (89,2). heap=[(10,0),(8,1)].
> - i=3: d=1, push (1,3). heap top=(10,0). size 3 > 2 → pop (10,0). heap=[(8,1),(1,3)].
>
> Final heap contains indices 1 and 3 → `[[-2,2], [0,1]]`. Their distances² are 8 and 1 — correct, the 2 closest.
>
> Notice the farthest point `[5,8]` got kicked immediately, and `[1,3]` (d=10) got kicked later when `[0,1]` (d=1) came in."

### 6. Complexity (20s)

> "**Time O(n log K)** — n insertions, each O(log K). **Space O(K)** for the heap. Contrast with sort's O(n log n) — when K is much smaller than n, heap wins."

### 7. Optimizations + follow-ups (1 min)

> "A few things I'd flag:
>
> 1. **Overflow**: if coordinates could be ~10⁵, then `x² + y²` might exceed 32-bit int (2×10¹⁰). I'd switch to `long long`. LC's constraints are usually safe, but worth naming out loud.
>
> 2. **Faster: quickselect / `nth_element`**. Expected O(n), worst O(n²) without random pivot. One line in C++:
>
>    ```cpp
>    nth_element(points.begin(), points.begin() + k, points.end(),
>                [](auto& a, auto& b) { return dist(a) < dist(b); });
>    return {points.begin(), points.begin() + k};
>    ```
>
>    Same trade-off as Kth Largest (0215) — heap is stable and streamable, quickselect is faster in the average case.
>
> 3. **Streaming variant**: if points arrive one at a time and we want the K closest so far, the heap approach adapts trivially — same code without the outer for. That's the 'Kth Largest in a Stream' pattern (0703 style).
>
> Any follow-ups you want me to code out?"

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
            priority_queue<pair<int, int>> pq;                       // max-heap
            for (int i = 0; i < (int)points.size(); i++) {
                int d = points[i][0] * points[i][0]
                      + points[i][1] * points[i][1];                 // 距离², 免开根
                pq.push({d, i});
                if ((int)pq.size() > k) pq.pop();                    // 踢最远
            }
            vector<vector<int>> res;
            while (!pq.empty()) {
                auto [_, i] = pq.top(); pq.pop();
                res.push_back(points[i]);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    import heapq

    class Solution:
        def kClosest(self, points: list[list[int]], k: int) -> list[list[int]]:
            # Python heapq 是 min-heap. 为了模拟 max-heap of k, 存 (-d, i)
            # 顶就是"负最大" = "距离最大", 踢它就是踢最远
            heap: list[tuple[int, int]] = []
            for i, (x, y) in enumerate(points):
                d = x * x + y * y
                heapq.heappush(heap, (-d, i))
                if len(heap) > k:
                    heapq.heappop(heap)
            return [points[i] for _, i in heap]

        # 一行 Pythonic — heapq.nsmallest 内部就是本题的 max-heap of k
        def kClosest_oneliner(self, points, k):
            return heapq.nsmallest(k, points, key=lambda p: p[0]**2 + p[1]**2)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} points
     * @param {number} k
     * @return {number[][]}
     */
    var kClosest = function(points, k) {
        // JS 无原生 heap. 对小数据直接 sort 也过 — 简洁
        // 面试严谨要手写堆或用 priority queue lib
        return points
            .sort((a, b) => (a[0]*a[0] + a[1]*a[1]) - (b[0]*b[0] + b[1]*b[1]))
            .slice(0, k);
    };
    ```

## Complexity

- **Time**: O(n log K) — 每点 push/pop 各 O(log K).
- **Space**: O(K) — 堆.

## 相关题目

- [0347. Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) — 频次 Top-K, 同款模板
- [0215. Kth Largest Element in an Array](../0215-kth-largest-element-in-an-array/README.md) — Top-K 单值, min-heap + quickselect
- [1046. Last Stone Weight](../1046-last-stone-weight/README.md) — max-heap 模拟
- [0239. Sliding Window Maximum](../0239-sliding-window-maximum/README.md) — 单调队列
- [0703. Kth Largest Element in a Stream](../0703-kth-largest-element-in-a-stream/README.md) — 流式版
- [0692. Top K Frequent Words](../0692-top-k-frequent-words/README.md) — 频次 + 字典序 tiebreak
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — 矩阵 + 堆
- 0295\. Find Median from Data Stream (待补) — 双堆维护中位数
- 0658\. Find K Closest Elements (待补) — 排序数组 + 二分定位 + 窗口
