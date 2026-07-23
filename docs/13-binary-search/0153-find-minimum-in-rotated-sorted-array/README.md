# 0153. Find Minimum in Rotated Sorted Array / 寻找旋转排序数组中的最小值

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search, Rotated Array · 二分查找, 旋转数组
    - **Link**: [LeetCode](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min in rotated sorted (no duplicates)** → **compare `nums[mid]` with `nums[r]`** (not `nums[l]`): if `mid ≤ r` right half is sorted → min in left inclusive (`r = mid`); else in right (`l = mid + 1`). Loop until `l == r`.
>
> **中文**: **旋转排序数组求最小 (无重复)** → **对比 `nums[mid]` 和 `nums[r]`** (不是 l): mid ≤ r 说明右半升序 → 最小在**左半含 mid** (`r = mid`); 否则右半 (`l = mid + 1`). 直到 l == r.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给一次**旋转**过的升序数组 (无重复). 找最小值.

- 例: `[3, 4, 5, 1, 2]` → 1 (原 `[1, 2, 3, 4, 5]` 左旋 3 次).

**中文**: 旋转升序数组求最小.

## Key Insights

1. **🔑 灵魂洞察: 旋转数组 = 两段升序 + 一个"断点" / Rotated ⇔ two sorted halves + one drop**

    ```
    原:  1 2 3 4 5           断点 = 从 5 → 1 的位置
    旋: 3 4 5 | 1 2
              ↑ 最小值在此 (断点右)
    ```

    - **未旋转 (整体升序)**: 断点在开头, 最小 = `nums[0]`.
    - **旋转过**: 断点在中间, 最小 = 断点右侧第一个元素.

    → 二分**找断点**.

2. **🔑 灵魂对比: `nums[mid]` vs `nums[r]` (不是 `nums[l]`) / Compare with r, not l**

    为啥必须跟**右端 `nums[r]`** 比?

    ```
    [3, 4, 5, 1, 2]  (旋转过)
    mid = 2, nums[mid] = 5
    对比 nums[r] = 2: 5 > 2 → 说明 [mid, r] 跨越了断点, 最小在右半 ✅
    对比 nums[l] = 3: 5 > 3 → 无法判断 (整体有序 [1,2,3,4,5] mid 也 > l)
    ```

    - **`nums[mid] ≤ nums[r]`** ⇒ **右半段 `[mid, r]` 是升序** (无断点) → 最小在**左半含 mid** (`r = mid`).
    - **`nums[mid] > nums[r]`** ⇒ **断点在 `(mid, r]`** → 最小在**右半** (`l = mid + 1`).

    > **右端 `nums[r]` 是"参考锚点"** — 它是右半段的最大值. mid 跟它比可以精准区分"跨没跨断点".

3. **🔑 `r = mid` 不 `r = mid - 1` / Update `r` to mid (inclusive)**

    因为 **mid 本身可能是最小值**. 例:

    ```
    [3, 1, 2]  mid = 1, nums[mid] = 1 (最小!), nums[r] = 2
    nums[mid] ≤ nums[r] → 若 r = mid - 1 = 0 → 丢失最小!
    正确: r = mid = 1, 保留候选
    ```

    > **"mid 可能是答案 → 不能剔除"** 是变体二分的通用规律. 什么时候 `mid - 1` 什么时候 `mid` **看语义**.

4. **🔑 循环条件 `l < r` (不是 `l <= r`) / Loop until l == r**

    因为最后我们**只保留 1 个元素** = 答案. `l == r` 时**就是那一个**, 无需再判.

    - **`l <= r`** 版本可以写但需要 `if (nums[mid] == target) return`, 本题**目标不是 target 匹配** 而是**找断点**, 所以用 `l < r` 更自然.

    > **二分模板 3 种边界**: `[l, r]` (l ≤ r + return mid), `[l, r)` (l < r + return l/r), `[l, r+1)` (少见). Yang 用**半开** `[l, r]` + `l < r`.

5. **🔑 返 `nums[l]` (等价 `nums[r]`) / Return nums[l]**

    结束时 `l == r`, 两者是同一位置. 返谁都行. Yang 返 `nums[l]`.

6. **🔑 已排序 (无旋转) 情况自动处理 / Handles non-rotated case**

    ```
    [1, 2, 3, 4, 5]  从未旋转
    mid = 2, nums[mid] = 3 ≤ nums[r] = 5 → r = mid
    继续: mid = 1, nums[mid] = 2 ≤ nums[r] = 3 → r = mid
    最后 l = r = 0, 返 nums[0] = 1 ✅
    ```

    **无需特判**, 二分自然把 r 一路推到最左. 优雅.

7. **🔑 有重复怎么办? → 0154 Find Minimum II / Duplicates variant**

    若允许重复 (`[2, 2, 2, 0, 1, 2]`):

    - **`nums[mid] == nums[r]`** 时无法判断 → **`r--` 慢缩** (worst O(n)).

    > **0154** 是本题的进阶, 加一个 `else if (nums[mid] == nums[r]) r--`. LC 上"II" 版都是这类"允许重复" 的变体.

8. **🔑 复杂度 O(log n) 时间, O(1) 空间 / Log time**

    每轮排除一半. 空间常数.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findMin(vector<int>& nums) {
            int l = 0, r = nums.size() - 1;
            while (l < r) {                              // l < r, 结束时 l == r
                int mid = l + (r - l) / 2;               // 防溢出
                if (nums[mid] <= nums[r]) {
                    r = mid;                              // 右半升序 → 最小在左半 (含 mid)
                } else {
                    l = mid + 1;                          // 断点在右半
                }
            }
            return nums[l];                              // l == r, 返谁都行
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findMin(self, nums: list[int]) -> int:
            l, r = 0, len(nums) - 1
            while l < r:
                mid = (l + r) // 2      # Python int 无溢出
                if nums[mid] <= nums[r]:
                    r = mid
                else:
                    l = mid + 1
            return nums[l]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var findMin = function(nums) {
        let l = 0, r = nums.length - 1;
        while (l < r) {
            const mid = Math.floor((l + r) / 2);
            if (nums[mid] <= nums[r]) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }
        return nums[l];
    };
    ```

## Complexity

- **Time**: O(log n) — 每轮排除一半.
- **Space**: O(1).

## 相关题目

- [0704. Binary Search](../0704-binary-search/README.md) — 标准二分母题
- [0074. Search a 2D Matrix](../0074-search-a-2d-matrix/README.md) — 一次二分 + 坐标映射
- [0875. Koko Eating Bananas](../0875-koko-eating-bananas/README.md) — BSA
- [2560. House Robber IV](../2560-house-robber-iv/README.md) — BSA
- 0154\. Find Minimum in Rotated Sorted Array II (待补) — **允许重复** 版, 加 `r--`
- [0033. Search in Rotated Sorted Array](../0033-search-in-rotated-sorted-array/README.md) — 在旋转数组里找 target
- 0081\. Search in Rotated Sorted Array II (待补) — 允许重复的找 target
- 0162\. Find Peak Element (待补) — 二分找峰值
- 0852\. Peak Index in a Mountain Array (待补) — 二分找山顶
