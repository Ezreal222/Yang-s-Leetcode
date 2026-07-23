# 0033. Search in Rotated Sorted Array / 搜索旋转排序数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search, Rotated Array · 二分查找, 旋转数组
    - **Link**: [LeetCode](https://leetcode.com/problems/search-in-rotated-sorted-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find target in rotated sorted array (no dup)** → binary search; each step **decide which half is sorted** (compare `nums[mid]` with `nums[right]`), then **check if `target` lies in that sorted half's range** — go into it if yes, else the other.
>
> **中文**: **旋转排序数组找 target** → 二分; 每步**判哪半是升序** (对比 mid vs right), 再**判 target 是否在升序段的值域**内 — 是就走该段, 否则走另一段.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 旋转过的升序数组 (无重复), 找 target 下标. 找不到返 -1. **O(log n)**.

**中文**: 旋转数组找 target, 返下标, 无返 -1.

## Key Insights

1. **🔑 灵魂洞察: 每次 mid 后, 至少一半升序 / At each step, at least one half is sorted**

    旋转数组 = 两段升序 + 一个断点. 任取 mid 划开成两半, **至少一半跨过 (含) 不了断点** → 那半**升序**.

    - **左半 `[l, mid]` 升序** ⇔ `nums[l] ≤ nums[mid]` (跟 [0153](../0153-find-minimum-in-rotated-sorted-array/README.md) 呼应, 但这里判 l, 也可判 nums[mid] > nums[r]).
    - **右半 `[mid, r]` 升序** ⇔ `nums[mid] ≤ nums[r]`.

    Yang 用 `nums[mid] > nums[right]` 判定 → **左半升序**; 反之右半升序.

    > **"两段中至少一段有序"** 是旋转数组二分的核心公理.

2. **🔑 灵魂第二步: 判 target 是否在**升序半**的范围内 / Does target lie in sorted half's range?**

    找到升序半后, **它的值域 = `[nums[段左], nums[段右]]`** 是明确区间. 判 target 是否落在这**明确区间**里:

    - **左半升序** (`nums[mid] > nums[r]`):
        - 若 **`nums[l] ≤ target < nums[mid]`** → target 在左半升序段 → `r = mid - 1`.
        - 否则 → target 在右半 → `l = mid + 1`.
    - **右半升序** (`nums[mid] ≤ nums[r]`):
        - 若 **`nums[mid] < target ≤ nums[r]`** → target 在右半升序段 → `l = mid + 1`.
        - 否则 → target 在左半 → `r = mid - 1`.

    > **明确升序段的边界 → target 是否在段内 → 排除另一段**. 这是从**部分有序**恢复到**普通二分** 的桥梁.

3. **🔑 Yang 的写法解读 / Yang's code annotated**

    ```cpp
    if (nums[mid] > nums[right]) {                      // 左半升序
        if (nums[mid] > target && nums[left] <= target) {   // target 在左半?
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    } else {                                            // 右半升序
        if (nums[mid] < target && nums[right] >= target) {  // target 在右半?
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    ```

    - 顶层判**升序半在哪** (跟 nums[right] 比).
    - 内层判**target 在不在**升序半.
    - 落在升序半 → 移动**远端**指针 (右边界左移 / 左边界右移) 进入.
    - 不在升序半 → 走另一半.

4. **🔑 二分模板: `<=` 闭区间 + `mid ± 1` / Closed interval template**

    - **`left <= right`**: 因为找具体 target, 允许 mid == 答案时**返** — 闭区间常用.
    - **`nums[mid] == target return mid`**: 一开始就判.
    - **`right = mid - 1, left = mid + 1`**: mid 已判过, 直接剔除.

    > 跟 [0153](../0153-find-minimum-in-rotated-sorted-array/README.md) 找断点用 `l < r + r = mid` 区别: **本题目标是值匹配**, 一旦命中直接返, 不需要 `mid` 保留.

5. **🔑 未旋转数组自动适配 / Non-rotated case handled**

    ```
    nums = [1, 2, 3, 4, 5], 从未旋转
    → nums[mid] ≤ nums[right] 恒成立 → 一直进"右半升序" 分支
    → 普通二分完成
    ```

    **不需要特判**. 旋转次数 = 0 时逻辑一致.

6. **🔑 跟 [0153 Find Minimum](../0153-find-minimum-in-rotated-sorted-array/README.md) 的对比 / vs 0153**

    | | **0033 (本题)** | 0153 |
    |---|---|---|
    | 找什么 | target 的下标 | 最小值 |
    | 关键比较 | mid vs right + 段内值域 | mid vs right |
    | 迭代边界 | `mid ± 1` (可剔除 mid) | `r = mid` (mid 可能是最小, 不剔) |
    | 循环条件 | `<=` | `<` |

    > 一族两题, 找法不同但**"用 nums[right] 判哪半升序"** 的思想同源.

7. **🔑 有重复变体: 0081 / Duplicates → 0081**

    `nums[mid] == nums[right]` 时**无法判**哪半升序 (0153 里也提过, 加 `right--` 慢缩). worst O(n).

    > **0081 是本题的"允许重复" 版本**, 加**一行** `else if (nums[mid] == nums[r]) r--`.

8. **🔑 复杂度 O(log n) 时间, O(1) 空间 / Log time**

    每步排除一半. 空间常数.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int search(vector<int>& nums, int target) {
            int left = 0, right = nums.size() - 1;
            while (left <= right) {
                int mid = (left + right) / 2;
                if (nums[mid] == target) return mid;
                if (nums[mid] > nums[right]) {                          // 左半升序
                    if (nums[mid] > target && nums[left] <= target) {   // target 在左半
                        right = mid - 1;
                    } else {
                        left = mid + 1;
                    }
                } else {                                                // 右半升序
                    if (nums[mid] < target && nums[right] >= target) {  // target 在右半
                        left = mid + 1;
                    } else {
                        right = mid - 1;
                    }
                }
            }
            return -1;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def search(self, nums: list[int], target: int) -> int:
            l, r = 0, len(nums) - 1
            while l <= r:
                mid = (l + r) // 2
                if nums[mid] == target:
                    return mid
                if nums[mid] > nums[r]:              # 左半升序
                    if nums[l] <= target < nums[mid]:
                        r = mid - 1
                    else:
                        l = mid + 1
                else:                                # 右半升序
                    if nums[mid] < target <= nums[r]:
                        l = mid + 1
                    else:
                        r = mid - 1
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
        let l = 0, r = nums.length - 1;
        while (l <= r) {
            const mid = Math.floor((l + r) / 2);
            if (nums[mid] === target) return mid;
            if (nums[mid] > nums[r]) {
                if (nums[l] <= target && target < nums[mid]) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else {
                if (nums[mid] < target && target <= nums[r]) {
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return -1;
    };
    ```

## Complexity

- **Time**: O(log n) — 每步排除一半.
- **Space**: O(1).

## 相关题目

- [0153. Find Minimum in Rotated Sorted Array](../0153-find-minimum-in-rotated-sorted-array/README.md) — 旋转数组找最小
- [0704. Binary Search](../0704-binary-search/README.md) — 标准二分母题
- [0074. Search a 2D Matrix](../0074-search-a-2d-matrix/README.md) — 一次二分 + 坐标映射
- [0875. Koko Eating Bananas](../0875-koko-eating-bananas/README.md) — BSA
- 0081\. Search in Rotated Sorted Array II (待补) — **允许重复**, 加 `r--`
- 0154\. Find Minimum in Rotated Sorted Array II (待补) — 允许重复找最小
- 0162\. Find Peak Element (待补) — 二分找峰值
- 0004\. Median of Two Sorted Arrays (待补) — 二分找中位数 Hard
