# 1094. Car Pooling / 拼车

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Prefix Sum, Sweep Line · 数组, 前缀和, 扫描线 (差分数组)
    - **Link**: [LeetCode](https://leetcode.com/problems/car-pooling/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `trips[i] = [numPassengers, from, to]`. Picking passengers up at `from` and dropping off at `to`. Car capacity is `capacity`. Return whether **all trips can be completed without exceeding capacity** at any point.

**中文**: `trips[i] = [人数, 上车点, 下车点]`. 车容量 `capacity`. 判断是否能在任意时刻都不超载地完成所有行程.

## Key Insights

1. **差分数组 (diff array) + 前缀和 / Diff array + prefix sum**

    跟 [0253 Meeting Rooms II](../../09-greedy/0253-meeting-rooms-ii/README.md) 是同款"任意时刻最大并发数" 的问题, 不过这里**坐标是位置, 不是时间**, 而且坐标已经离散化在 `[0, 1000]` (LC 约束).

    模板:

    ```
    diff[from] += people;          // 上车: +people
    diff[to]   -= people;          // 下车: -people
    prefix sum 一遍扫, 任何时刻 > capacity → false
    ```

    `O(N + M)` (N = trips, M = 坐标范围). 比 heap 解 (O(N log N)) 更快 — 因为坐标范围小, 直接铺 diff 数组.

2. **关键细节: `to` 减号位置 / Why subtract at `to`, not `to+1`**

    本题语义是"`to` 是下车点 — 到 `to` 时乘客**就下车了**". 所以 `[from, to)` 是乘客在车上的区间, **`to` 点本身不算乘客在车上**.

    因此:

    - `diff[from] += people`: 从 from **开始** 增加.
    - `diff[to]   -= people`: 从 to **开始** 减少 (即 to 处的前缀和已经把这批人扣掉了).

    若题目改成"到 `to` 点还在车上, 之后才下", 应写 `diff[to+1] -= people`.

3. **跟 [0253](../../09-greedy/0253-meeting-rooms-ii/README.md) 的对比 / vs Meeting Rooms II**

    | 题 | 坐标 | 求什么 | 选哪个解 |
    |---|---|---|---|
    | **1094 (本题)** | 位置 [0, 1000] (小, 整数) | 任意时刻 ≤ capacity? | **diff 数组** (坐标小, 优势) |
    | 0253 会议室 | 时间 (无界) | 任意时刻最大并发 | **heap** 或 **离散化扫描线** |

    > **坐标范围小 + 整数** → 直接 diff 数组. 范围大或浮点 → 必须 heap / 离散化.

4. **vs 扫描线写法 / vs sweep-line variant**

    扫描线写法是把事件 `(pos, ±people)` 全收集起来再 sort + 扫. 本题坐标已经是数组下标, **省了 sort** — diff 数组就是"用下标天然排好序" 的扫描线. 一句话: **diff 数组 = 当坐标小且离散时的扫描线快捷写法**.

5. **差分数组的通用模板 / General diff-array pattern**

    `[l, r)` 区间整体 +v: `diff[l] += v; diff[r] -= v;` (左闭右开).

    `[l, r]` 闭区间整体 +v: `diff[l] += v; diff[r+1] -= v;`.

    扫一遍前缀和复原每个位置的值. 用于"多次区间加, 最后一次性查询每点值" 的场景, O(N+M) 不是 O(NM).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool carPooling(vector<vector<int>>& trips, int capacity) {
            vector<int> diff(1001, 0);                             // LC 约束: 0 <= from < to <= 1000
            for (auto& t : trips) {
                diff[t[1]] += t[0];                                // 上车: +people
                diff[t[2]] -= t[0];                                // 下车 (to 点已经下车) → -people
            }
            int cur = 0;
            for (int d : diff) {
                cur += d;                                          // 前缀和 = 当前位置车上人数
                if (cur > capacity) return false;
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def carPooling(self, trips: list[list[int]], capacity: int) -> bool:
            # [0] * 1001: 等价 C++ 的 vector<int>(1001, 0)
            diff = [0] * 1001
            for people, frm, to in trips:                          # 三元组解包, frm 避开关键字 from
                diff[frm] += people
                diff[to]  -= people
            cur = 0
            # 也可以 itertools.accumulate(diff) 一行算前缀和, 但显式 for 更对照 C++
            for d in diff:
                cur += d
                if cur > capacity:
                    return False
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} trips
     * @param {number} capacity
     * @return {boolean}
     */
    var carPooling = function(trips, capacity) {
        // new Array(1001).fill(0): 一行铺零数组, 等价 vector<int>(1001, 0)
        const diff = new Array(1001).fill(0);
        for (const [people, from, to] of trips) {                  // 数组解构, 等价 Python 的元组解包
            diff[from] += people;
            diff[to]   -= people;
        }
        let cur = 0;
        for (const d of diff) {
            cur += d;
            if (cur > capacity) return false;
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(N + M), N = trips, M = 1001 (坐标范围).
- **Space**: O(M) — diff 数组.

## 相关题目

- [0253. Meeting Rooms II](../../09-greedy/0253-meeting-rooms-ii/README.md) — 同款"任意时刻最大并发", 用 heap / 扫描线 (坐标无界版)
- 0218\. The Skyline Problem (待补) — 扫描线 + heap 经典
- 0732\. My Calendar III (待补) — 同款"任意时刻并发数"
- 0598\. Range Addition II (待补) — 二维差分入门
- [0370. Range Addition](../0370-range-addition/README.md) — 一维差分模板题, 闭区间版
- [1109. Corporate Flight Bookings](../1109-corporate-flight-bookings/README.md) — 同款差分数组, 1-indexed 闭区间
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 对偶: 前缀和模板 (差分的逆运算)
