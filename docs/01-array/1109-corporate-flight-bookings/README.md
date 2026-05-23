# 1109. Corporate Flight Bookings / 航班预订统计

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Prefix Sum · 数组, 前缀和 (差分数组)
    - **Link**: [LeetCode](https://leetcode.com/problems/corporate-flight-bookings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `n` flights numbered `1..n`. `bookings[i] = [first, last, seats]` means: on every flight in **1-indexed** closed range `[first, last]`, `seats` are booked. Return an array of length `n` — total seats booked on each flight.

**中文**: `n` 个航班, 编号 `1..n` (**1-indexed**). `bookings[i] = [first, last, seats]` 表示在 `[first, last]` 闭区间内每个航班都加了 `seats` 个座位. 返回长度 `n` 的数组, 每个航班最终的总座位数.

## Key Insights

1. **直接套差分数组模板 / Straight diff-array template**

    跟 [0370 Range Addition](../0370-range-addition/README.md) 同款, 差别只在**输入是 1-indexed**. 把"`[first, last]` 整体 +seats" 翻译到 0-indexed 闭区间 `[first-1, last-1]`, 然后:

    ```
    diff[first - 1] += seats;        // 0-indexed start
    diff[last]      -= seats;        // 0-indexed (end-1)+1 = last
    ```

    最后前缀和复原.

2. **1-indexed → 0-indexed 的 -1 偏移 / 1-based to 0-based offset**

    给 +1: `first - 1` 是把 1-indexed 起点转 0-indexed 起点.

    减 -1: 因为闭区间 `[first-1, last-1]` 的"右端 +1" 是 `last`. 写成 `diff[last]` 一行带过, 不用再 -1 +1.

    > 一句话: **闭区间差分公式 `diff[r+1] -= v` 中, 这里的 r = last - 1, 所以 r+1 = last**. 老套路.

3. **diff 数组大小: `n + 1` 给哨兵 / Size `n + 1` for the sentinel**

    最坏 `last = n`, 则 `diff[n] -= s`. 数组要开 `n + 1` 避免越界. 最后 `res` 只取 `diff[0..n-1]` 的前缀和, **`diff[n]` 是哨兵, 不读**.

4. **跟 [1094 拼车](../1094-car-pooling/README.md) / [0370](../0370-range-addition/README.md) 的关系 / Sister problems**

    三道题本质同一个模板, 只是包装不同:

    | 题 | 输入语义 | 0-index 公式 |
    |---|---|---|
    | **1109 (本题)** | 1-indexed 闭区间 `[first, last]` | `diff[first-1] += s; diff[last] -= s` |
    | 0370 | 0-indexed 闭区间 `[start, end]` | `diff[start] += inc; diff[end+1] -= inc` |
    | 1094 拼车 | 0-indexed 半开 `[from, to)` | `diff[from] += p; diff[to] -= p` |

    > **判别 trick**: "减号位置 - 起点位置 = 区间内有几个位置". 1109 是 `last - (first-1) = last - first + 1`, 正确; 0370 是 `(end+1) - start = end - start + 1`, 正确.

5. **为什么不能"暴力嵌套循环" / Why naive O(k·n) hurts**

    `k = 2e4`, `n = 2e4`, 暴力 4e8 操作, LC 必 TLE. 差分把每次操作 O(n) 压成 O(1), 总 O(k + n) ≈ 4e4, 秒过.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> corpFlightBookings(vector<vector<int>>& bookings, int n) {
            vector<int> diff(n + 1, 0);                            // +1 哨兵, 给 diff[n] 留槽
            for (auto& b : bookings) {
                diff[b[0] - 1] += b[2];                            // 1-indexed first → 0-indexed: -1
                diff[b[1]]     -= b[2];                            // 闭区间右端 last → last+1-1 = last
            }
            vector<int> res(n);
            res[0] = diff[0];
            for (int i = 1; i < n; i++) {
                res[i] = res[i - 1] + diff[i];                     // 前缀和复原
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from itertools import accumulate

    class Solution:
        def corpFlightBookings(self, bookings: list[list[int]], n: int) -> list[int]:
            diff = [0] * (n + 1)                                   # +1 哨兵
            for first, last, seats in bookings:                    # 解包三元组
                diff[first - 1] += seats                           # 1-indexed → 0-indexed: -1
                diff[last]      -= seats                           # 闭区间右端+1
            # accumulate 一行算前缀和, 切片去掉哨兵那格
            return list(accumulate(diff))[:n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} bookings
     * @param {number} n
     * @return {number[]}
     */
    var corpFlightBookings = function(bookings, n) {
        const diff = new Array(n + 1).fill(0);
        for (const [first, last, seats] of bookings) {             // 数组解构 + for...of
            diff[first - 1] += seats;
            diff[last]      -= seats;
        }
        const res = new Array(n);
        res[0] = diff[0];
        for (let i = 1; i < n; i++) {
            res[i] = res[i - 1] + diff[i];
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(k + n), k = bookings.
- **Space**: O(n) — diff + res.

## 相关题目

- [0370. Range Addition](../0370-range-addition/README.md) — 同款差分, 0-indexed 闭区间版
- [1094. Car Pooling](../1094-car-pooling/README.md) — 同款差分, 半开区间版
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 对偶: 前缀和模板
- 1893\. Check if All the Integers in a Range Are Covered (待补) — 差分 + 区间覆盖判定
- 0598\. Range Addition II (待补) — 二维差分简化版
- 2848\. Points That Intersect With Cars (待补) — 差分 + 集合大小
