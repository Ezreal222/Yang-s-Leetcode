# 0875. Koko Eating Bananas / 爱吃香蕉的珂珂

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search on Answer (BSA), Greedy Check · 二分答案, 贪心校验
    - **Link**: [LeetCode](https://leetcode.com/problems/koko-eating-bananas/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min eating speed to finish in h hours** → **BSA on speed `k ∈ [1, max(piles)]`**; check `t = Σ ceil(p / k) ≤ h`. Higher k ⇒ easier → **monotonic** predicate. Return leftmost k where check passes.
>
> **中文**: **h 小时内吃完的最小速度** → **BSA 在 `k ∈ [1, max(piles)]`**; check `t = Σ ⌈p / k⌉ ≤ h`. k 越大越易通过 → **单调**. 返最小可行 k.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: `piles[i]` = 第 i 堆香蕉的数量. 珂珂每小时选 **1 堆** 吃 **k 根** (堆里不够 k 也就吃完). 求让她**在 h 小时内吃完** 的**最小 k**.

**中文**: 最小速度 h 小时吃完所有堆.

## Key Insights

1. **🔑 灵魂洞察: 这是 BSA (二分答案) / Binary Search on Answer**

    直接对 k 二分**答案本身**. 判定谓词:

    - **can(k)** = 用速度 k 能否 h 小时内吃完?
    - **单调**: **k 越大** → 每堆花时间越少 → **can(k) 越易为真**.
    - → 存在**分界点** k*, `can(k) = false, k < k*`, `can(k) = true, k ≥ k*`. **k* 就是答案**.

    > **BSA 三件套**: ① 答案在某区间 ② 单调判定 ③ check 函数. 见 [2560](../2560-house-robber-iv/README.md) / [1011](../1011-capacity-to-ship-packages-within-d-days/README.md) / [1231](../1231-divide-chocolate/README.md).

2. **🔑 搜索范围: `[1, max(piles)]` / Search bounds**

    - **`left = 1`**: 最慢, 每小时吃 1 根.
    - **`right = max(piles)`**: **一小时能吃完最大堆** — 再大也没意义 (每小时最多吃一堆).

    > `right = 10^9` 无脑上界也对, 但**紧界**加快搜索. 用 `*max_element` 求.

3. **🔑 check 函数: `⌈p / k⌉` 求每堆用时 / Ceiling division**

    一堆 p 根, 速度 k → 用**`⌈p / k⌉`** 小时 (最后一小时可能吃不满 k).

    ```cpp
    long long t = 0;
    for (int p : piles) t += (p + mid - 1) / mid;    // ceil(p/mid)
    ```

    **`(p + k - 1) / k` = ⌈p / k⌉**: 整数除法上取整招式.

    - **`p / k`** 是下取整.
    - 加上 `k - 1`: 让所有非零余数**进位** 到下一个整数.
    - 除完就是**上取整**.

    > **这个 ceil 技巧**在 LC 里烂大街. 记住 `(a + b - 1) / b` 是 `⌈a / b⌉` (a, b 都非负).

4. **🔑 `long long` 防溢出 / Overflow guard**

    Yang 用 `long long mid`, `long long t`. 因为:

    - `p + mid - 1` 若 p, mid 都到 1e9, 加起来 ~2e9, 超 int (2.1e9 上限).
    - `t = Σ ⌈p / k⌉` 累加也可能超. n × 1e9 就爆.

    > **数值题第一反应查溢出**. 上界 ≥ 1e5 时看看是否需 `long long`. 老手必查.

5. **🔑 二分模板: `left <= right` 闭区间 / Closed interval template**

    ```cpp
    while (left <= right) {
        long long mid = left + (right - left) / 2;
        if (can(mid))   right = mid - 1;             // 可行 → 尝试更小
        else            left = mid + 1;              // 不可行 → 更大
    }
    return left;                                     // 循环结束 left = 最小可行
    ```

    - **返 `left`**: 循环结束时 `left = right + 1`, `left` **越过了所有不可行的 k**, 就是**最小可行**.
    - **不返 `mid`**: mid 是循环内的临时, 结束时无意义.

    > **返 left / right 的口诀**: 找**最小可行** 返 `left`, 找**最大可行** 返 `right`. 记熟就不会返错.

6. **🔑 复杂度 O(n log M) 时间, O(1) 空间 / Log × linear**

    - **Time**: 二分 log(max_pile) 次 × 每次 check O(n) = **O(n log M)**.
    - **Space**: O(1) 指针.

7. **🔑 跟 [1011 Ship Capacity](../1011-capacity-to-ship-packages-within-d-days/README.md) 是**双胞胎** / Twin of 1011**

    | | **0875 (本题)** | 1011 |
    |---|---|---|
    | 答案 | 最小吃速 | 最小载重 |
    | check | ⌈p/k⌉ 累加 ≤ h | 装箱累加 ≤ D 天 |
    | 结构 | 完全一样 | 完全一样 |

    > **"最小/最大化 + 单调可行" = BSA**. 换个包装还是同一题.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minEatingSpeed(vector<int>& piles, int h) {
            int left = 1, right = *max_element(piles.begin(), piles.end());
            while (left <= right) {
                long long mid = left + (right - left) / 2;
                long long t = 0;
                for (int p : piles) t += (p + mid - 1) / mid;        // ceil(p/mid)
                if (t > h) left = mid + 1;                            // 太慢, 加速
                else       right = mid - 1;                           // 够快, 试更小
            }
            return left;
        }
    };
    ```

=== "Python"
    ```python
    import math

    class Solution:
        def minEatingSpeed(self, piles: list[int], h: int) -> int:
            left, right = 1, max(piles)
            while left <= right:
                mid = (left + right) // 2       # Python int 无溢出, //  即整除
                # sum + genexp + math.ceil 一行算总时间
                # 也可 -(-p // mid) 是纯整数 ceil 技巧 (双负号翻转下取整)
                t = sum(math.ceil(p / mid) for p in piles)
                if t > h:
                    left = mid + 1
                else:
                    right = mid - 1
            return left
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} piles
     * @param {number} h
     * @return {number}
     */
    var minEatingSpeed = function(piles, h) {
        // Math.max(...piles) — spread 展开, 但大数组会 stack overflow. 用 reduce 稳
        // let right = piles.reduce((a, b) => Math.max(a, b), 0);
        let left = 1, right = Math.max(...piles);
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            // Math.ceil(p / mid) — JS Number 是 double, 除法直接算浮点再 ceil
            // 数据范围到 2^53 内 (LC 保证) 精度无损, 不用 BigInt
            let t = 0;
            for (const p of piles) t += Math.ceil(p / mid);
            if (t > h) left = mid + 1;
            else right = mid - 1;
        }
        return left;
    };
    ```

## Complexity

- **Time**: O(n log M) — M = max(piles).
- **Space**: O(1).

## 相关题目

- [0704. Binary Search](../0704-binary-search/README.md) — 一维二分母题
- [0074. Search a 2D Matrix](../0074-search-a-2d-matrix/README.md) — 一次二分 + 坐标映射
- [1011. Capacity To Ship Packages Within D Days](../1011-capacity-to-ship-packages-within-d-days/README.md) — **BSA 双胞胎**
- [2560. House Robber IV](../2560-house-robber-iv/README.md) — BSA + 贪心 check
- [1231. Divide Chocolate](../1231-divide-chocolate/README.md) — BSA + 最小值最大化
- 0410\. Split Array Largest Sum (待补) — BSA + 分段
- 0774\. Minimize Max Distance to Gas Station (待补) — BSA + 浮点
- 0668\. Kth Smallest Number in Multiplication Table (待补) — BSA + 乘法表
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — BSA + 矩阵
- 2064\. Minimized Maximum of Products Distributed to Any Store (待补) — BSA + 分配
