# 0977. Squares of a Sorted Array / 有序数组的平方

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, Array, Sorting · 双指针, 数组, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/squares-of-a-sorted-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: Sorted array with negatives squared → max values are at **both ends** → opposite-direction two pointers compare `|abs|`, larger goes to result tail.
>
> **中文**: 升序 (含负) 平方后, **最大值在两端** → 对撞双指针比绝对值, 大的塞结果末尾.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给**非递减** 数组 `nums` (可能含**负数**). 返回**每个元素平方后** 的**非递减** 数组.

**中文**: 升序数组 (有负), 返回平方后的升序数组.

## Key Insights

1. **🔑 对撞双指针 — 跟 [0027 同向双指针](../0027-remove-element/README.md) 形成两大模式 / Opposite-direction two pointers**

    跟 0027 / 0727 的"快慢双指针" (同向) 不同, 本题用**两端往中间夹** 的对撞模式:

    | | 同向 (0027) | **对撞 (本题)** |
    |---|---|---|
    | 指针位置 | 都从左开始 | **一左一右** |
    | 推进方向 | 同方向 (慢追快) | **相向而行 (合拢)** |
    | 终止 | `i >= n` | **`i > j`** (交叉) |

    > 双指针两大流派: **同向追** vs **对撞合**. 看到"有序" / "对称比较" → 反应对撞.

2. **🔑 平方后最大值在两端 — 不是中间 / Largest squares at the ends**

    原数组升序: 负数 → 0 → 正数. 平方后:

    - **最负的元素 (最左)** 平方后**很大**.
    - **最正的元素 (最右)** 平方后**也很大**.
    - **最小的元素 (绝对值最小, 可能在中间)** 平方后**最小**.

    所以**两个最大平方在两端** → 用对撞双指针, 每次比两端的**绝对值** 取大的填进结果末尾.

    > **理解题目结构**比写代码重要. 看清"平方后哪里最大" 直接锁定算法.

3. **🔑 倒序填结果数组 / Fill result from the back**

    既然每次拿"剩余里平方最大的", 自然**从结果末尾往前填** — `k = n - 1, n - 2, ..., 0`.

    Yang 用 `res[k--]`: 后置 `--` 先用 k 当下标, 再 k 减 1. 等价 `res[k] = ...; k--;`.

4. **🔑 比较 `abs(nums[i])` 跟 `abs(nums[j])` / Compare absolute values**

    平方比较等价绝对值比较 (非负数下), 但**用 abs 更直观** — 避免"两个负数相乘是正" 的脑回路绕路. Yang 用 abs.

5. **`<=` 不是 `<` 处理两端相同 / `<=` for tie-breaking**

    `while (i <= j)` 让 i 和 j 重合时也处理一次 — 最后剩的那个元素也要放进 res. 写 `<` 会漏中间最小那个.

    > 双指针的终止条件 (`<` vs `<=`) 跟具体题型相关. **写错就漏一个或多算一个**.

6. **复杂度 O(n) 时间, O(n) 空间 / Linear**

    一遍扫, 不能原地 (要保留 nums 的顺序). 跟"先平方再排序" 的 O(n log n) 比快.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> sortedSquares(vector<int>& nums) {
            int i = 0, j = nums.size() - 1, k = nums.size() - 1;
            vector<int> res(nums.size());
            while (i <= j) {                                       // <= 不是 <, 处理 i == j
                if (abs(nums[i]) >= abs(nums[j])) {
                    res[k--] = nums[i] * nums[i];                  // 左端绝对值大, 填末尾
                    i++;
                } else {
                    res[k--] = nums[j] * nums[j];                  // 右端绝对值大
                    j--;
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sortedSquares(self, nums: list[int]) -> list[int]:
            n = len(nums)
            i, j, k = 0, n - 1, n - 1
            res = [0] * n
            while i <= j:
                if abs(nums[i]) >= abs(nums[j]):
                    res[k] = nums[i] ** 2
                    i += 1
                else:
                    res[k] = nums[j] ** 2
                    j -= 1
                k -= 1
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number[]}
     */
    var sortedSquares = function(nums) {
        const n = nums.length;
        let i = 0, j = n - 1, k = n - 1;
        const res = new Array(n);
        while (i <= j) {
            if (Math.abs(nums[i]) >= Math.abs(nums[j])) {
                res[k--] = nums[i] * nums[i];
                i++;
            } else {
                res[k--] = nums[j] * nums[j];
                j--;
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍对撞扫.
- **Space**: O(n) — 结果数组. (题目要求新数组, 不能原地)

## 相关题目

- [0027. Remove Element](../0027-remove-element/README.md) — 同向双指针母题
- [0727. Minimum Window Subsequence](../0727-minimum-window-subsequence/README.md) — 同向双指针 + 回溯
- 0167\. Two Sum II - Input Array Is Sorted (待补) — 对撞双指针经典
- 0011\. Container With Most Water (待补) — 对撞双指针 + 贪心
- [0015. 3Sum](../0015-3sum/README.md) — 排序 + 对撞双指针 + 3 层去重
- 0344\. Reverse String (待补) — 对撞最简单
- [0042. Trapping Rain Water](../../11-monotonic-stack/0042-trapping-rain-water/README.md) — 对撞双指针 O(1) 空间
