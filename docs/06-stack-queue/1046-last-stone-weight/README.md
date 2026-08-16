# 1046. Last Stone Weight / 最后一块石头的重量

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Heap, Priority Queue, Simulation · 堆, 优先队列, 模拟
    - **Link**: [LeetCode](https://leetcode.com/problems/last-stone-weight/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Repeatedly smash two heaviest stones (larger − smaller), until ≤ 1 left** → **max-heap**: pop 2 heaviest, push their diff if > 0. Return remaining top or 0.
>
> **中文**: **反复取两个最重的相撞 (大 − 小), 直到 ≤ 1 个** → **最大堆**: 弹两个, 差值 > 0 再入堆. 返堆顶或 0.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 每步取**两个最重** 的石头 x ≥ y 相撞:

- x == y → 都碎.
- x > y → x 变成 x - y.

反复到 ≤ 1 块, 返最后剩的重量 (无剩返 0).

**中文**: 反复取最重两个相撞, 求最后剩下.

## Key Insights

1. **🔑 灵魂: 每步要动态"最大值" → 反射 max-heap / Dynamic max → max-heap**

    每步都要**当前最重的两个**. 若排序 + 每步取两 + 差值插回, 差值插入需重排 → **O(n² log n)**.

    **max-heap**: pop 2 顶各 O(log n), push 1 个 O(log n) → **每步 O(log n)**, 总 **O(n log n)**.

    > **"频繁增删 + 动态最值"** = heap 天生适用. 见到就反射.

2. **🔑 C++ `priority_queue<int>` 默认 max-heap / Default is max-heap in C++**

    ```cpp
    priority_queue<int> pq;              // max-heap (默认)
    priority_queue<int, vector<int>, greater<int>> pq;  // min-heap
    ```

    - **默认 max**: 因为 `less<int>` 是"a<b 则 a 优先级低 → a 沉底".
    - **min-heap 显式**: `greater<>` 反过来.

    > **跟 [0215 Kth Largest](../0215-kth-largest-element-in-an-array/README.md) / [0347 Top-K](../0347-top-k-frequent-elements/README.md) 用 min-heap 相反**. 本题要"最大", 直接默认.

3. **🔑 只在 `newStone > 0` 才 push / Push only if > 0**

    ```cpp
    int newStone = s1 - s2;
    if (newStone > 0) pq.push(newStone);
    ```

    - `s1 == s2` (等重) → 都碎, **不加**新石.
    - `s1 > s2` → 加 s1 - s2.

    若 `push(0)` 也不算错, 但**多一次无用 push/pop** — 优化.

4. **🔑 循环条件 `pq.size() > 1` / Loop while ≥ 2 stones**

    每次消耗 2 个, 可能加回 1 个 (净 -1) 或不加 (净 -2). 循环到剩 ≤ 1 为止.

    - 剩 1 个 → 返堆顶.
    - 剩 0 个 → 返 0.

    Yang 用 `pq.empty() ? 0 : pq.top()` 一行兜.

5. **🔑 复杂度 O(n log n) 时间, O(n) 空间 / Complexity**

    - Time: 每次 pop/push O(log n), 循环最多 n 次.
    - Space: 堆最多 n 个石.

6. **🔑 跟 [1049 Last Stone Weight II](../../10-dp/1049-last-stone-weight-ii/README.md) 的对比 / vs 1049**

    | | **1046 (本题)** | 1049 |
    |---|---|---|
    | 规则 | 每次取**最重**两个 (贪心) | **随意配对** 求最小差 |
    | 解法 | max-heap 模拟 | **0/1 背包 DP** |
    | 难度 | Easy | Medium |

    > **一字之差, 差一个量级**. "最重两个" (贪心) vs "任意两个" (DP 全搜) 天壤之别.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int lastStoneWeight(vector<int>& stones) {
            priority_queue<int> pq;                                  // max-heap
            for (int x : stones) pq.push(x);
            while (pq.size() > 1) {
                int s1 = pq.top(); pq.pop();                         // 最重
                int s2 = pq.top(); pq.pop();                         // 次重
                int newStone = s1 - s2;
                if (newStone > 0) pq.push(newStone);                 // > 0 才回堆
            }
            return pq.empty() ? 0 : pq.top();
        }
    };
    ```

=== "Python"
    ```python
    import heapq

    class Solution:
        def lastStoneWeight(self, stones: list[int]) -> int:
            # Python heapq 只有 min-heap. 存负值模拟 max-heap
            # heapify 原地 O(n) 建堆, 比一个个 heappush 快
            heap = [-x for x in stones]
            heapq.heapify(heap)
            while len(heap) > 1:
                s1 = -heapq.heappop(heap)       # 取正值
                s2 = -heapq.heappop(heap)
                if s1 > s2:
                    heapq.heappush(heap, -(s1 - s2))
            return -heap[0] if heap else 0
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} stones
     * @return {number}
     */
    var lastStoneWeight = function(stones) {
        // JS 无原生 heap. 小数据 (LC 保证 n ≤ 30) 直接每次排序也 O(n²) 可过
        // 严谨面试要手写 max-heap 或用 lib
        while (stones.length > 1) {
            stones.sort((a, b) => b - a);       // 降序
            const s1 = stones.shift();
            const s2 = stones.shift();
            if (s1 > s2) stones.push(s1 - s2);
        }
        return stones.length ? stones[0] : 0;
    };
    ```

## Complexity

- **Time**: O(n log n) — n 次 pop/push.
- **Space**: O(n) — 堆.

## 相关题目

- [0347. Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) — min-heap of k
- [0215. Kth Largest Element in an Array](../0215-kth-largest-element-in-an-array/README.md) — min-heap of k / quickselect
- [0239. Sliding Window Maximum](../0239-sliding-window-maximum/README.md) — 单调队列 (另一种"动态最值")
- [1049. Last Stone Weight II](../../10-dp/1049-last-stone-weight-ii/README.md) — **一字之差, DP 版**
- 0703\. Kth Largest Element in a Stream (待补) — 流式版
- [0973. K Closest Points to Origin](../0973-k-closest-points-to-origin/README.md) — max-heap of k, distance² 优化
- 0295\. Find Median from Data Stream (待补) — 双堆维护中位数
- 0692\. Top K Frequent Words (待补) — 频次 + 字典序
