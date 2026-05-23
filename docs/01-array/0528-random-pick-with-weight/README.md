# 0528. Random Pick with Weight / 按权重随机选择

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Prefix Sum, Binary Search, Random, Design · 数组, 前缀和, 二分查找, 随机, 设计
    - **Link**: [LeetCode](https://leetcode.com/problems/random-pick-with-weight/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given positive int array `w`. Implement `pickIndex()` returning index `i` with probability `w[i] / sum(w)`.

**中文**: 给正整数数组 `w`. 实现 `pickIndex()`, 返回下标 `i` 的概率为 `w[i] / sum(w)`.

## Key Insights

1. **核心: 前缀和 + 在 [1, total] 上均匀采样 + 二分查找 / Inverse CDF sampling**

    把 `w` 看作每个下标的"区间长度", 在数轴上首尾相接铺出区间:

    ```
    w = [1, 3, 2]
    prefix = [1, 4, 6]
    数轴段: [1,1]  [2,4]  [5,6]
            └ i=0  └ i=1  └ i=2 (长度 = w[i])
    ```

    在 `[1, total]` 上**均匀**取一个 target → 落到哪段, 答案就是哪个下标. 各段长 = `w[i]` 保证概率正比于权重.

    要找"落在哪段" → 找**第一个 `prefix[i] >= target`** → 二分 (lower_bound 模板).

2. **`rand() % total + 1` 让 target ∈ [1, total] / Pick target in [1, total]**

    `rand() % total` 给 `[0, total-1]`. `+1` 推到 `[1, total]`.

    为什么不要 0? 因为 `prefix[0] = w[0] >= 1`. 若 `target = 0`, 任何 `prefix[i] >= 0` 都成立, 二分会返回 i=0 — 但 i=0 应该只对应**长度 w[0]** 的概率, 多吃了一个数会破坏均匀性.

    等价写法: `target = rand() % total` (target ∈ [0, total-1]), 二分找**第一个 `prefix[i] > target`** (upper_bound). 区别只是边界包含哪一头.

3. **lower_bound 二分模板 / Standard lower_bound binary search**

    Yang 的写法是教科书 lower_bound:

    ```
    while left < right:
        mid = (left + right) / 2;
        if prefix[mid] < target:  left = mid + 1   // mid 太小, 排除 mid
        else:                     right = mid       // mid 可能就是答案, 不能 -1
    return left;
    ```

    > 三点要记: ① `right = mid` 不是 `mid - 1` (要保留 mid); ② 循环条件 `left < right` 严格小于; ③ `mid = left + (right - left) / 2` 防溢出 (虽然这题 int 不会爆).

    C++ STL: `lower_bound(prefix.begin(), prefix.end(), target) - prefix.begin()` 一行替代.

4. **构造 O(n), 每次 pickIndex O(log n) / Build O(n), query O(log n)**

    一次预处理换多次 O(log n) 查询. 比"每次重新 sample" 的 O(n) 强很多.

    > 进阶: **Alias method (别名采样)** 能做到构造 O(n) + 每次 O(1) 查询, 但实现复杂 6 倍, LC 这题没必要.

5. **跟其它前缀和题的位置 / Where it sits**

    | 题 | 前缀和的作用 |
    |---|---|
    | [0303 区间和](../0303-range-sum-query-immutable/README.md) | 区间和 O(1) 查 |
    | [0304 二维区间和](../0304-range-sum-query-2d-immutable/README.md) | 二维区间和 O(1) 查 |
    | [1094](../1094-car-pooling/README.md) / [0370](../0370-range-addition/README.md) / [1109](../1109-corporate-flight-bookings/README.md) | 差分 → 前缀和复原 |
    | **0528 (本题)** | **前缀和定义"累积概率分布", 二分逆向采样** |

    > 一句话: 前缀和把"和的累积" 物化成数组, 想做什么取决于你怎么用它 — 查区间和? 查累积? 还是反查"哪段"?

## Solution

=== "C++"
    ```cpp
    class Solution {
        vector<int> prefix;
        int total;
    public:
        Solution(vector<int>& w) {
            prefix.resize(w.size());
            prefix[0] = w[0];
            for (int i = 1; i < (int)w.size(); i++) {
                prefix[i] = prefix[i - 1] + w[i];                  // 累积权重
            }
            total = prefix.back();
        }
        int pickIndex() {
            int target = rand() % total + 1;                       // target ∈ [1, total]
            int left = 0, right = prefix.size() - 1;
            while (left < right) {                                 // lower_bound: 找第一个 prefix[i] >= target
                int mid = left + (right - left) / 2;
                if (prefix[mid] < target) left = mid + 1;          // mid 太小, 排除
                else                       right = mid;             // mid 可能就是答案, 保留
            }
            return left;
        }
    };
    ```

=== "Python"
    ```python
    import random
    from bisect import bisect_left

    class Solution:
        def __init__(self, w: list[int]):
            # accumulate(w): 一行算前缀和 (无初始 0); 这里需要 prefix[0] = w[0], 所以不加 initial
            from itertools import accumulate
            self.prefix = list(accumulate(w))
            self.total  = self.prefix[-1]

        def pickIndex(self) -> int:
            # random.randint(a, b): 闭区间 [a, b], 等价 C++ 的 rand() % total + 1
            target = random.randint(1, self.total)
            # bisect_left = 找第一个 >= target 的位置, 等价手写 lower_bound. 比手写二分 3 倍快且不会写错
            return bisect_left(self.prefix, target)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} w
     */
    var Solution = function(w) {
        this.prefix = new Array(w.length);
        this.prefix[0] = w[0];
        for (let i = 1; i < w.length; i++) {
            this.prefix[i] = this.prefix[i - 1] + w[i];
        }
        this.total = this.prefix[this.prefix.length - 1];
    };

    /**
     * @return {number}
     */
    Solution.prototype.pickIndex = function() {
        // Math.random() ∈ [0, 1), * total → [0, total), Math.floor 取整 → [0, total-1], +1 → [1, total]
        const target = Math.floor(Math.random() * this.total) + 1;
        let left = 0, right = this.prefix.length - 1;
        while (left < right) {                                     // lower_bound 模板
            const mid = (left + right) >> 1;                       // >>1 等价 /2 取整, 比 Math.floor 快
            if (this.prefix[mid] < target) left = mid + 1;
            else                            right = mid;
        }
        return left;
    };
    ```

## Complexity

- **Time**: 构造 O(n); 每次 `pickIndex` O(log n).
- **Space**: O(n) — prefix 数组.

## 相关题目

- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和基础
- 0497\. Random Point in Non-overlapping Rectangles (待补) — 同款"前缀和 + 二分采样", 矩形面积版
- 0710\. Random Pick with Blacklist (待补) — 另一类"加权随机" 设计
- 0398\. Random Pick Index (待补) — 蓄水池采样入门
- 0382\. Linked List Random Node (待补) — 蓄水池采样链表版
- 0875\. Koko Eating Bananas (待补) — 同款 lower_bound 二分模板用法
