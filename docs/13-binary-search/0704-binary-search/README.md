# 0704. Binary Search / 二分查找

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Binary Search, Array · 二分查找, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-search/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给**升序** 数组 `nums` 和目标值 `target`. 返回 `target` 在数组中的下标, 不在则返 `-1`. 必须 O(log n).

**中文**: 有序数组找 target 的下标, 不在返 -1, 要 O(log n).

## Key Insights

1. **🔑 二分模板 — 整个 §13 的基石 / The foundation of all §13 problems**

    本题是 §13 [Binary Search](../index.md) 的入门. **死磕一种区间约定**, 后面 [2560](../2560-house-robber-iv/README.md) / [1011](../1011-capacity-to-ship-packages-within-d-days/README.md) / [1231](../1231-divide-chocolate/README.md) 的二分答案 BSA 全是它的变种.

2. **🔑 两种区间约定: `[L, R]` 闭 vs `[L, R)` 半开 / Two conventions**

    Yang 用**半开** `[left, right)`. 两种约定的对应规则:

    | | 闭区间 `[L, R]` | 半开 `[L, R)` (Yang) |
    |---|---|---|
    | 初始 `right` | `n - 1` | **`n`** |
    | while 条件 | `L <= R` | **`L < R`** |
    | `nums[mid] > target` | `R = mid - 1` | **`R = mid`** |
    | `nums[mid] < target` | `L = mid + 1` | **`L = mid + 1`** |
    | "区间为空" | `L > R` | **`L == R`** |

    > **挑一个用到底**. 混着写最容易死循环或越界. 半开约定跟 STL `lower_bound` / Python `bisect` 同, 后面 BSA / 二分边界题用得更顺.

3. **🔑 `mid = left + (right - left) / 2` 防溢出 / Overflow-safe midpoint**

    朴素 `(left + right) / 2` 在大数据 (`left + right` 接近 `INT_MAX`) 时会**溢出**. **`left + (right - left) / 2`** 等价但永不溢出.

    > LC 数据下 `int` 通常够, 但**养成习惯写防溢出版本**. 工业代码必备.

4. **🔑 更新规则: 找到等于直接返, 否则缩到不包含 mid 的子区间 / Shrink to subrange without mid**

    每次判断后, **mid 这个位置在下一轮区间外** (我们知道它不是答案). 所以:

    - `nums[mid] > target` → 答案在左半, `R = mid` (右开, 排除 mid).
    - `nums[mid] < target` → 答案在右半, `L = mid + 1` (左闭, 排除 mid).

    这样每轮区间**严格缩小**, 不死循环.

5. **终止时 `L == R` 是空区间 / Loop exits at empty interval**

    `L == R` 时 `[L, R)` 长度 0, 没有候选 → 没找到, 返 -1. **是 Yang return -1 那一行能成立的原因**.

6. **复杂度 O(log n) / 经典对数**

    每轮区间至少减半, 最多 `⌈log₂ n⌉` 轮. 空间 O(1).

## Solution

=== "C++"
    === "v1 (Yang 原版): 半开区间 `[L, R)`"
        ```cpp
        class Solution {
        public:
            int search(vector<int>& nums, int target) {
                int left = 0, right = nums.size();                 // 半开: right = n
                while (left < right) {                             // 区间非空
                    int mid = left + (right - left) / 2;           // 防溢出
                    if (nums[mid] == target) return mid;
                    if (nums[mid] > target) right = mid;           // 排除 mid
                    else                    left = mid + 1;
                }
                return -1;
            }
        };
        ```

    === "v2: 闭区间 `[L, R]` (备选)"
        ```cpp
        class Solution {
        public:
            int search(vector<int>& nums, int target) {
                int left = 0, right = (int)nums.size() - 1;        // 闭: right = n-1
                while (left <= right) {                            // <= 不是 <
                    int mid = left + (right - left) / 2;
                    if (nums[mid] == target) return mid;
                    if (nums[mid] > target) right = mid - 1;       // 排除 mid
                    else                    left = mid + 1;
                }
                return -1;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def search(self, nums: list[int], target: int) -> int:
            left, right = 0, len(nums)                             # 半开 [L, R)
            while left < right:
                mid = (left + right) // 2                          # Python int 无溢出风险
                if nums[mid] == target:
                    return mid
                elif nums[mid] > target:
                    right = mid
                else:
                    left = mid + 1
            return -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number}
     */
    var search = function(nums, target) {
        let left = 0, right = nums.length;                         // 半开
        while (left < right) {
            const mid = (left + right) >> 1;                       // 位运算除 2 (正数)
            if (nums[mid] === target) return mid;
            if (nums[mid] > target) right = mid;
            else                    left = mid + 1;
        }
        return -1;
    };
    ```

## Complexity

- **Time**: O(log n).
- **Space**: O(1).

## 相关题目

- 0035\. Search Insert Position (待补) — 找插入位置 (lower_bound)
- 0034\. Find First and Last Position of Element in Sorted Array (待补) — 找元素首尾位置
- 0033\. Search in Rotated Sorted Array (待补) — 旋转有序数组 + 二分
- 0153\. Find Minimum in Rotated Sorted Array (待补) — 旋转数组找最小
- 0278\. First Bad Version (待补) — 找第一个 true (BSA 入门)
- [2560. House Robber IV](../2560-house-robber-iv/README.md) — 二分答案 (BSA) 应用
- [1011. Capacity To Ship Packages Within D Days](../1011-capacity-to-ship-packages-within-d-days/README.md) — BSA "最小化最大值"
- [1231. Divide Chocolate](../1231-divide-chocolate/README.md) — BSA "最大化最小值"
