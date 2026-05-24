# 1755. Closest Subsequence Sum / 最接近目标值的子序列和

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Meet in the Middle, Bitmask, Sort, Binary Search · 折半搜索, 状压, 排序, 二分
    - **Link**: [LeetCode](https://leetcode.com/problems/closest-subsequence-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给数组 `nums` (n ≤ 40, 值可正可负) 和整数 `goal`. 选一个**子序列** (可以为空) 让其和与 `goal` 的差的绝对值最小, 返回这个最小绝对差.

**中文**: 给 `nums` (n ≤ 40, 值可正可负) 和 `goal`, 选一个子序列让 `|sum - goal|` 最小.

## Key Insights

1. **纯折半模板 — 没"个数约束" / Vanilla Meet-in-the-Middle**

    跟 [2035](../2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) 同思路, 但少了"必须各选 n 个" 的约束 → **不用按 popcount 分桶**.

    | 项 | 2035 (带约束) | 1755 (纯模板) |
    |---|---|---|
    | 数据结构 | `vector<vector<int>>` 按 popcount 分桶 | 单个 `vector<int>` |
    | 配对 | 左选 k + 右选 n-k | 任意组合 |
    | 代码量 | 多一层 | 短一截 |

    > **纯模板更适合入门折半** — 把基础打牢再加约束.

2. **n ≤ 40 的暗示 / The n=40 fingerprint**

    `2^40 ≈ 10^12` 必爆, 但 `2^(40/2) = 2^20 ≈ 10^6` 完全可以. 看到 `n ≤ 40` + 子序列 + 普通 DP 困难 → 立刻反应折半.

    > 详见 [§10 · 折半搜索 (Meet in the Middle)](../topic-meet-in-the-middle.md).

3. **空子序列 = mask=0 = 自动包含 / Empty subseq covered for free**

    题目允许空子序列 (和为 0). bitmask 枚举 `mask = 0` 时 `sum = 0` 自然落进数组, 不用特判.

4. **二分: lower_bound + 两侧候选 / Binary search + check both neighbors**

    排好 `rightSums`. 对每个 `sumL`, 想要 `sumR ≈ goal - sumL`. `lower_bound` 给出第一个 ≥ 目标的位置, 最接近的可能就是它或它前一个 → `d ∈ {-1, 0}` 都试.

    > 二分"找最接近" 的标配 — 找到分界点后两侧都要看.

5. **`vector<int>(n+1)` 还是 `vector<int>{}` ? — 别多初始化 / Don't pre-size the result vector**

    Yang 原代码:

    ```cpp
    vector<int> leftSums(n + 1);   // ⚠ 创建 n+1 个 0 填进去!
    ...
    leftSums.push_back(sum);       // 之后又 push_back
    ```

    `vector<int> v(n+1)` 创建**含 n+1 个零** 的 vector, 不是预留空间. 之后 `push_back` 在 0 后面追加 → 数组里多出 `n+1` 个虚假 0. 程序**碰巧不会错** (空子序列和也是 0, 跟正常 mask=0 重复), 但浪费时间 + 概念混乱.

    正确两种写法:

    ```cpp
    vector<int> leftSums;                  // 空 vector, 边 push 边扩
    // 或
    vector<int> leftSums;
    leftSums.reserve(1 << n);              // 真正"预留容量" (避免扩容拷贝)
    ```

    > `vector<T>(n)` = 装 n 个默认值, `vector<T>` + `reserve(n)` = 空但预留. 别混.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minAbsDifference(vector<int>& nums, int goal) {
            int n = nums.size() / 2;
            int m = nums.size() - n;                               // 处理奇数长度

            vector<int> leftSums, rightSums;                       // 注意: 不要 (n+1) 预填 0
            leftSums.reserve(1 << n);
            rightSums.reserve(1 << m);

            for (int mask = 0; mask < (1 << n); mask++) {
                int sum = 0;
                for (int i = 0; i < n; i++) {
                    if (mask & (1 << i)) sum += nums[i];
                }
                leftSums.push_back(sum);
            }
            for (int mask = 0; mask < (1 << m); mask++) {
                int sum = 0;
                for (int i = 0; i < m; i++) {
                    if (mask & (1 << i)) sum += nums[n + i];
                }
                rightSums.push_back(sum);
            }

            sort(rightSums.begin(), rightSums.end());

            int result = INT_MAX;
            for (int sumL : leftSums) {
                int targetR = goal - sumL;
                int idx = lower_bound(rightSums.begin(), rightSums.end(), targetR)
                          - rightSums.begin();
                for (int d = -1; d <= 0; d++) {                    // 二分两侧都看
                    int i = idx + d;
                    if (i >= 0 && i < (int)rightSums.size()) {
                        result = min(result, abs(sumL + rightSums[i] - goal));
                    }
                }
            }
            return result;
        }
    };
    ```

=== "Python"
    ```python
    from bisect import bisect_left

    class Solution:
        def minAbsDifference(self, nums: list[int], goal: int) -> int:
            n = len(nums) // 2
            m = len(nums) - n                                      # 处理奇数长度

            def all_subset_sums(arr):
                # 列表推导 + 生成器: 对 2^k 个 mask 计算和
                # 等价 C++ 双层 for
                k = len(arr)
                return [
                    sum(arr[i] for i in range(k) if mask & (1 << i))
                    for mask in range(1 << k)
                ]

            left_sums = all_subset_sums(nums[:n])
            right_sums = sorted(all_subset_sums(nums[n:]))         # 排序为二分准备

            result = float('inf')
            for sl in left_sums:
                target_r = goal - sl
                # bisect_left = C++ lower_bound, 返回首个 ≥ target 的索引
                idx = bisect_left(right_sums, target_r)
                for d in (-1, 0):
                    i = idx + d
                    if 0 <= i < len(right_sums):
                        result = min(result, abs(sl + right_sums[i] - goal))
            return result
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} goal
     * @return {number}
     */
    var minAbsDifference = function(nums, goal) {
        const n = nums.length >> 1, m = nums.length - n;

        const allSubsetSums = (arr) => {
            const k = arr.length, out = [];
            for (let mask = 0; mask < (1 << k); mask++) {
                let s = 0;
                for (let i = 0; i < k; i++) {
                    if (mask & (1 << i)) s += arr[i];
                }
                out.push(s);
            }
            return out;
        };

        const leftSums = allSubsetSums(nums.slice(0, n));
        // sort 默认字典序, 数字必须传 compareFn (a-b) — 不传会把 -1, 2, 10 排成 -1,10,2
        const rightSums = allSubsetSums(nums.slice(n)).sort((a, b) => a - b);

        const lowerBound = (arr, target) => {
            let lo = 0, hi = arr.length;
            while (lo < hi) {
                const mid = (lo + hi) >> 1;
                if (arr[mid] < target) lo = mid + 1;
                else hi = mid;
            }
            return lo;
        };

        let result = Infinity;
        for (const sl of leftSums) {
            const idx = lowerBound(rightSums, goal - sl);
            for (const d of [-1, 0]) {
                const i = idx + d;
                if (i >= 0 && i < rightSums.length) {
                    result = Math.min(result, Math.abs(sl + rightSums[i] - goal));
                }
            }
        }
        return result;
    };
    ```

## Complexity

- **Time**: O(2^(n/2) × n) — 枚举 (内层 O(n)) + 排序/二分 O(2^(n/2) × n/2). n=40: `2^20 × 20 ≈ 2×10^7`.
- **Space**: O(2^(n/2)) — 两个子集和数组.

## 相关题目

- [§10 DP · 折半搜索 (Meet in the Middle) 技巧总结](../topic-meet-in-the-middle.md) — 母方法
- [2035. Partition Array Into Two Arrays to Minimize Sum Difference](../2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) — 折半 + popcount 分桶 (带"各选 n 个" 约束)
- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 同 "凑最接近 total/2" 思想, 但用 0/1 背包 (n 大值小)
- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 同模型, 判定相等
- 956\. Tallest Billboard (待补) — 同 "分两堆" 思想, 三状态 DP 或折半
