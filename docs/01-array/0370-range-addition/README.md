# 0370. Range Addition / 区间加法

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Prefix Sum · 数组, 前缀和 (差分数组)
    - **Link**: [LeetCode](https://leetcode.com/problems/range-addition/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Length-`length` array starts all-0. Apply `k` operations `updates[i] = [start, end, inc]` — each adds `inc` to every element in **closed** range `[start, end]`. Return the final array.

**中文**: 长度为 `length` 的数组, 初始全 0. 执行 `k` 个操作 `[start, end, inc]` — 把闭区间 `[start, end]` 的每个元素加 `inc`. 返回最终数组.

## Key Insights

1. **差分数组模板题: 闭区间版 / Diff array template (closed-interval flavor)**

    朴素做法每次操作 O(end - start + 1), 总 O(k·n). 差分数组把每次操作压成 **O(1)** 两次写, 最后一次 O(n) 前缀和复原, 总 O(k + n).

    闭区间 `[start, end]` 整体 +inc 的差分公式:

    ```
    diff[start]   += inc;
    diff[end + 1] -= inc;            // ← 关键: end+1, 不是 end
    ```

    > 之所以 `end + 1`: 差分的"-inc" 表示**从这个位置开始不再加** — 我们希望 `end` 处仍然加, 所以从 `end + 1` 开始撤销.

2. **闭 vs 半开 (跟 [1094](../1094-car-pooling/README.md) 对比) / Closed vs half-open**

    | 题 | 区间语义 | 减号在哪 |
    |---|---|---|
    | **0370 (本题)** | 闭 `[l, r]` | `diff[r + 1] -= v` |
    | 1094 拼车 | 半开 `[l, r)` (`to` 是下车点) | `diff[r] -= v` |

    > 一句话: **想让位置 `i` 仍然加**, 减号就放 `i+1`. **想让位置 `i` 不再加** (端点已是出口), 减号放 `i`.

    本题 `diff` 开 `length + 1` 大小, 给"`end + 1` 可能 == length" 留个槽位 — 这块永远不被读回 res 里, 纯哨兵.

3. **复原: 前缀和 / Recover via prefix sum**

    最终值数组 = 差分数组的前缀和:

    ```
    res[0] = diff[0];
    for i in 1..n-1: res[i] = res[i-1] + diff[i];
    ```

    可以原地把 `diff` 当 `res` 用 (节省一个数组), 但分离写更清晰.

4. **差分数组的数学直觉 / Math intuition**

    把数组 `a` 写成"差分序列" `d[i] = a[i] - a[i-1]` (`d[0] = a[0]`). 然后 `a[i] = d[0] + d[1] + ... + d[i]` (前缀和).

    在 `[l, r]` 整体 +v, 等价于:

    - `d[l] += v` (l 处与前一位的差 +v).
    - `d[r+1] -= v` (r+1 处与 r 的差 -v, 抵消).
    - 中间所有 d 不变 (因为相邻差不变).

    一句话: **差分把"区间整体加" 变成两个端点的 O(1) 更新**, 再用前缀和回到原数组.

5. **何时用差分 / When diff array shines**

    - **多次区间整体加, 最后一次性查所有值** → 差分完美 (O(k + n)).
    - **多次区间整体加, 每次都要查某点值** → 差分仍可用, 但需要每次重算前缀和; 此时改用**树状数组** 更优.
    - **单点改, 区间求和** → 差分错配, 用普通前缀和或树状数组.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> getModifiedArray(int length, vector<vector<int>>& updates) {
            vector<int> diff(length + 1, 0);                       // +1 给 end+1 可能 == length 留槽位
            for (auto& u : updates) {
                diff[u[0]]     += u[2];                            // [start, end] +inc → diff[start] +inc
                diff[u[1] + 1] -= u[2];                            //                    → diff[end+1] -inc
            }
            vector<int> res(length);
            res[0] = diff[0];
            for (int i = 1; i < length; i++) {
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
        def getModifiedArray(self, length: int, updates: list[list[int]]) -> list[int]:
            diff = [0] * (length + 1)                              # +1 哨兵, 避免 end+1 越界
            for start, end, inc in updates:                        # 解包三元组
                diff[start]   += inc
                diff[end + 1] -= inc
            # accumulate(diff) 一行算前缀和, 然后切片到 length (去掉哨兵那一格)
            # list(...) 因为 accumulate 返回迭代器
            return list(accumulate(diff))[:length]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} length
     * @param {number[][]} updates
     * @return {number[]}
     */
    var getModifiedArray = function(length, updates) {
        const diff = new Array(length + 1).fill(0);
        for (const [start, end, inc] of updates) {                 // 数组解构 + for...of
            diff[start]     += inc;
            diff[end + 1]   -= inc;
        }
        const res = new Array(length);
        res[0] = diff[0];
        for (let i = 1; i < length; i++) {
            res[i] = res[i - 1] + diff[i];                         // 前缀和
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(k + n), k = updates, n = length.
- **Space**: O(n) — diff + res 数组.

## 相关题目

- [1094. Car Pooling](../1094-car-pooling/README.md) — 同款差分数组, 半开区间版
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 对偶: 前缀和模板 (差分的逆运算)
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- 1109\. Corporate Flight Bookings (待补) — 同款差分数组直接套
- 1893\. Check if All the Integers in a Range Are Covered (待补) — 差分 + 区间覆盖判定
- 0598\. Range Addition II (待补) — 二维差分简化版 (只关心左上角)
- 0307\. Range Sum Query - Mutable (待补) — 单点改 + 区间求和 → 树状数组 / 线段树
