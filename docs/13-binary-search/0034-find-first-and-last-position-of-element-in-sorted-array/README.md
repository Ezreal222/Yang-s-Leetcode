# 0034. Find First and Last Position of Element in Sorted Array / 在排序数组中查找元素的第一个和最后一个位置

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search, Lower Bound · 二分查找, 下界
    - **Link**: [LeetCode](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **First and last index of `target` in sorted array** → **`lower_bound(target)`** = first, **`lower_bound(target + 1) - 1`** = last. Guard: if `lower_bound(target)` out-of-range or `nums[it] != target` ⇒ `[-1, -1]`.
>
> **中文**: **有序数组找 target 首末位置** → **`lower_bound(target)`** = 首, **`lower_bound(target + 1) - 1`** = 末. 无 target 时返 `[-1, -1]`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 排序数组 `nums`, 找 `target` 的**首末**索引. 无返 `[-1, -1]`. **O(log n)**.

**中文**: 有序数组找 target 的第一个和最后一个位置.

## Key Insights

1. **🔑 灵魂招式: 一个 `lower_bound` 搞两端 / One helper handles both ends**

    - **`lower_bound(x)`**: 找**第一个 `>= x`** 的位置 (STL 语义).
    - **左边界** = `lower_bound(target)` (第一个 == target 的位置).
    - **右边界** = `lower_bound(target + 1) - 1` (第一个 > target 的位置 - 1 = 最后一个 == target).

    ```
    nums = [5,7,7,8,8,10], target = 8
    lower_bound(8) = 3 (第一个 8)
    lower_bound(9) = 5 (第一个 > 8), 减 1 = 4 (最后一个 8)
    return [3, 4]
    ```

    > **"一个 lower_bound 搞两端"** 是 STL 招式的美感. 比"左右各写一个二分" (需两种不同判定) 干净.

2. **🔑 `lower_bound` 模板 (半开 `[l, r)`) / lower_bound template with `[l, r)`**

    ```cpp
    int lowerBound(vector<int>& nums, int x) {
        int l = 0, r = nums.size();                      // 半开: r = size, 不是 size - 1
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] >= x) r = mid;                 // 满足, 尝试更左
            else l = mid + 1;
        }
        return l;
    }
    ```

    - **半开区间 `[l, r)`**: r 从 `size` 开始 (可返 size 表示"没找到").
    - **`nums[mid] >= x`**: 满足 → `r = mid` (mid 可能是答案, 保留); 不满足 → `l = mid + 1`.
    - 循环结束: `l == r`, 是**第一个 >= x** 的位置 (可能 = size).

    > **半开 `[l, r)` + `l < r` + 结束返 l** 是 lower_bound 的**标准模板**. 跟 [0981 找最大 ≤](../0981-time-based-key-value-store/README.md) 的闭区间模板互补, 各有语境.

3. **🔑 无 target 的两种情况 / Two "not found" cases**

    ```cpp
    int left = lowerBound(nums, target);
    if (left == nums.size() || nums[left] != target) return {-1, -1};
    ```

    - **`left == n`**: target 比所有元素**都大** → 数组里没.
    - **`nums[left] != target`**: 找到"第一个 >= target"但**不等于** target → 也没.

    **两个 check 都必要**, 缺一就 WA.

4. **🔑 为啥 `lower_bound(target + 1) - 1` 是右边界? / Why `lower_bound(target + 1) - 1` works**

    "第一个 > target" 的位置 = "第一个 >= target+1" 的位置 (因为整数序列).

    - 减 1 得**最后一个 == target** 的位置.
    - 若数组里**没 target**, 就是找到别的元素 — 但我们已经在**左边界** step 里排除了这种情况, 此时保证有 target.

    > **"上界 - 1 = 下界的右端"** 是 STL 惯用 trick — `upper_bound` 和 `lower_bound(target+1)` 是同义的 (整数).

5. **🔑 对比"手写两次二分" / vs manual two-binary-search version**

    朴素做法: 写两个不同的二分, 一个找首 (右缩), 一个找末 (左缩). **判定条件不对称**, 易错.

    Yang 的 `lower_bound + lower_bound(target+1) - 1` **统一模板**, 只写一份.

    > **"通用工具 + 参数变化"** 是干净代码的通用哲学.

6. **🔑 STL 原生等价 / STL native**

    C++ 里 `std::lower_bound(begin, end, x)` 就是这个 helper. Yang 手写了一份 (面试友好). 竞赛写:

    ```cpp
    auto left = lower_bound(nums.begin(), nums.end(), target);
    auto right = upper_bound(nums.begin(), nums.end(), target);
    if (left == nums.end() || *left != target) return {-1, -1};
    return {(int)(left - nums.begin()), (int)(right - nums.begin() - 1)};
    ```

    > **面试通常要手写** (不能靠 STL). 但两版都会 = 灵活.

7. **🔑 复杂度 O(log n) 时间, O(1) 空间 / Log time**

    - 两次 lower_bound = O(log n).
    - Space: O(1).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int lowerBound(vector<int>& nums, int x) {
            int l = 0, r = nums.size();
            while (l < r) {
                int mid = l + (r - l) / 2;
                if (nums[mid] >= x) r = mid;
                else l = mid + 1;
            }
            return l;
        }
        vector<int> searchRange(vector<int>& nums, int target) {
            int left = lowerBound(nums, target);
            if (left == (int)nums.size() || nums[left] != target) return {-1, -1};
            int right = lowerBound(nums, target + 1) - 1;
            return {left, right};
        }
    };
    ```

=== "Python"
    ```python
    from bisect import bisect_left

    class Solution:
        def searchRange(self, nums: list[int], target: int) -> list[int]:
            # bisect_left(a, x) = lower_bound(x) — 第一个 >= x 的位置
            # 跟 C++ std::lower_bound 语义一致
            left = bisect_left(nums, target)
            if left == len(nums) or nums[left] != target:
                return [-1, -1]
            # bisect_right(a, x) = upper_bound(x) = lower_bound(x + 1)
            # right = bisect_right(nums, target) - 1 也等价
            right = bisect_left(nums, target + 1) - 1
            return [left, right]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number[]}
     */
    var searchRange = function(nums, target) {
        // JS 无原生 lower_bound, 手写
        const lowerBound = (x) => {
            let l = 0, r = nums.length;
            while (l < r) {
                const mid = Math.floor((l + r) / 2);
                if (nums[mid] >= x) r = mid;
                else l = mid + 1;
            }
            return l;
        };
        const left = lowerBound(target);
        if (left === nums.length || nums[left] !== target) return [-1, -1];
        const right = lowerBound(target + 1) - 1;
        return [left, right];
    };
    ```

## Complexity

- **Time**: O(log n) — 两次二分.
- **Space**: O(1).

## 相关题目

- [0704. Binary Search](../0704-binary-search/README.md) — 标准二分母题
- [0074. Search a 2D Matrix](../0074-search-a-2d-matrix/README.md) — 一次二分 + 坐标映射
- [0033. Search in Rotated Sorted Array](../0033-search-in-rotated-sorted-array/README.md) — 旋转数组找 target
- [0153. Find Minimum in Rotated Sorted Array](../0153-find-minimum-in-rotated-sorted-array/README.md) — 找断点
- [0875. Koko Eating Bananas](../0875-koko-eating-bananas/README.md) — BSA 最小可行
- [0981. Time Based Key-Value Store](../0981-time-based-key-value-store/README.md) — 二分找最大 ≤
- 0035\. Search Insert Position (待补) — 直接就是 `lowerBound`
- 0278\. First Bad Version (待补) — 二分找边界
- 0532\. K-diff Pairs in an Array (待补) — 二分 + hash
- 0658\. Find K Closest Elements (待补) — 二分定位 + 窗口
