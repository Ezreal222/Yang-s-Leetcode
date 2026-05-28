# 0027. Remove Element / 移除元素

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, Array · 双指针, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-element/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 原地移除数组 `nums` 中所有等于 `val` 的元素, 返回新长度. 不允许额外开新数组, 元素顺序可改但前 `k` 个元素必须是保留的值.

**中文**: 原地删除等于 `val` 的元素, 返回新长度. O(1) 额外空间.

## Key Insights

1. **🔑 同向双指针: 快扫读, 慢写 / Fast/slow pointer in-place**

    这是双指针最基础的"同向" 模式. 两个指针:

    - **快指针 `i`**: 扫**每个**元素 (读).
    - **慢指针 `j`**: 只在"要保留" 时写, 写完前进.

    `i` 比 `j` 跑得快 (或相等). 扫完后 `j` 正好停在"新长度" 的位置.

    > **快慢同向**是双指针的两大模式之一 (另一是"对撞双指针"). 凡是"原地过滤 / 去重 / 压缩" 类题都套这个模板.

2. **状态: `nums[0..j) = 已保留的元素` / Invariant**

    任意时刻, `nums[0..j)` 都是已经"过滤" 完的合法前缀. 慢指针 `j` 是下一个待写位置.

3. **写入逻辑: 仅当 `nums[i] != val` 才写 / Write only when keeping**

    ```cpp
    if (nums[i] != val) nums[j++] = nums[i];
    ```

    `j++` 后置: 先写 `nums[j]`, 再 `j` 前进. 等元素时直接跳过 (i 自己进, j 不动).

    > 一行带后置 `++` 紧凑. 等价 `nums[j] = nums[i]; j++;` 两行版.

4. **返回 `j` — 不是 `i` / Return new length j**

    `j` 是新数组长度 (写了多少). `i` 是扫描位置 (等于原数组长度). 别返错.

5. **原地修改, O(1) 额外空间 / In-place, O(1) extra**

    所有写入都在原 `nums` 上, 不开新数组. 是双指针类题的核心优势.

6. **替代: 同款"判定 + 同向双指针" 通用模板 / Generalize to any predicate**

    把 `nums[i] != val` 换成任意保留条件 `keep(nums[i])`, 模板照搬:

    - 0026\. Remove Duplicates from Sorted Array (待补): 保留与左邻不同的.
    - 0283\. Move Zeroes (待补): 保留非零.
    - 0080\. Remove Duplicates II (待补): 保留出现不超过 2 次.

    > **快慢双指针 + 单条件过滤** 是数组类题的最常见 idiom.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int removeElement(vector<int>& nums, int val) {
            int j = 0;                                             // 慢指针: 下一个写入位置
            for (int i = 0; i < (int)nums.size(); i++) {           // 快指针: 扫每个元素
                if (nums[i] != val) {
                    nums[j++] = nums[i];                           // 保留, 写入后慢指针前进
                }
            }
            return j;                                              // 新长度
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def removeElement(self, nums: list[int], val: int) -> int:
            j = 0                                                  # 慢指针
            for i in range(len(nums)):                             # 快指针
                if nums[i] != val:
                    nums[j] = nums[i]
                    j += 1
            return j
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} val
     * @return {number}
     */
    var removeElement = function(nums, val) {
        let j = 0;
        for (let i = 0; i < nums.length; i++) {
            if (nums[i] !== val) {
                nums[j++] = nums[i];
            }
        }
        return j;
    };
    ```

## Complexity

- **Time**: O(n) — 快指针扫一遍.
- **Space**: O(1) — 原地.

## 相关题目

- 0026\. Remove Duplicates from Sorted Array (待补) — 同模板, 条件改为"跟前一个不同"
- 0080\. Remove Duplicates from Sorted Array II (待补) — 同模板, 条件"出现 ≤ 2 次"
- 0283\. Move Zeroes (待补) — 同模板, 非零保留, 末尾补零
- 0088\. Merge Sorted Array (待补) — 倒序双指针 + 合并
- 0844\. Backspace String Compare (待补) — 双指针倒序模拟退格
- 0011\. Container With Most Water (待补) — **对撞双指针** (另一类模式)
- [0042. Trapping Rain Water](../../11-monotonic-stack/0042-trapping-rain-water/README.md) — 对撞双指针经典
