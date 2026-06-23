# 0209. Minimum Size Subarray Sum / 长度最小的子数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, Sliding Window, Array · 双指针, 滑动窗口, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-size-subarray-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **All-positive array + want subarray sum ≥ target** → **variable-size sliding window**: expand right; while window sum ≥ target, record min length + shrink left.
>
> **中文**: **全正数 + 求子数组和 ≥ target** → **不定长滑动窗口**: 右扩, 满足时记长度并左缩.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给**正整数** 数组 `nums` 和正整数 `target`. 求**和 ≥ target** 的**最短连续子数组** 的长度. 没有返 `0`.

**中文**: 正数数组, 求和 ≥ target 的最短连续子数组长度, 没有返 0.

## Key Insights

1. **🔑 不定长滑动窗口 — 双指针的第三大模式 / Variable-size sliding window: TP's third major pattern**

    跟之前 [§05](../index.md) 的两类对照, 这是**第三种** 双指针模式:

    | 模式 | 代表题 | 形态 |
    |---|---|---|
    | 同向快慢 | [0027 移除元素](../0027-remove-element/README.md) | 慢指针写, 快指针读 |
    | 对撞合拢 | [0977 平方排序](../0977-squares-of-a-sorted-array/README.md) | 两端往中间夹 |
    | **不定长滑窗** (本题) | **0209** | **右扩 + 左缩, 维护"窗口属性"** |

    > **三大模式**: 同向 / 对撞 / 滑窗. 看清数据特征选哪种.

2. **🔑 滑窗模板: 右扩 + 内层 while 缩 / Expand right, shrink in while**

    Yang 的两层结构是滑窗的标准模板:

    ```
    for right in [0, n):
        加入 nums[right]                              // 右扩
        while 窗口满足条件:
            更新答案 (这里是 minLen)                   // 记
            移除 nums[left]; left++                   // 左缩
    ```

    - **for 外层**: 右指针**每步走 1**, 不会回退.
    - **while 内层**: 左指针**贪心缩**, 直到窗口刚好不满足.

3. **🔑 为什么内层是 `while` 不是 `if`? / Why while, not if**

    若用 `if`, 每次只缩 1 步 → 可能错过更短的窗口. 例: 一个超长元素一下子让窗口大幅超 target → 需要连续缩多个 left 才回到刚好满足. `while` 把缩短做到底.

    > **写错 `if` → WA**. 这是新手最常翻车的点.

4. **🔑 为什么是 "**正数**"? / Why all-positive matters**

    正数保证:

    - **右扩 → 窗口和单调不减** (新加正数, 和只会变大或不变).
    - **左缩 → 窗口和单调不增** (减去正数, 和只会变小或不变).

    这两个单调性让"滑窗一次扫" 成立 — 不需要回退. **若数组含负数 / 0**, 单调性破坏, 滑窗不适用, 需要前缀和 + 二分 / 单调队列 等其他招.

    > **看到"正数 + 连续子数组 + 求和"** → 立刻反应滑窗. 看到"可含负数" 警觉换招.

5. **每个元素入窗出窗各一次 → O(n) 摊销 / Amortized O(n)**

    left, right 各自最多走 n 步, 内层 while 总迭代数也是 O(n). **总 O(n)** — 比"枚举所有子数组" 的 O(n²) 快一阶.

6. **`INT_MAX` 哨兵 + 兜底返 0 / Sentinel + default 0**

    `minLen = INT_MAX` 初始. 若一次都没缩过 (整个数组和都 < target), 仍是 INT_MAX → 返 0.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minSubArrayLen(int target, vector<int>& nums) {
            int n = nums.size();
            int left = 0, sum = 0;
            int minLen = INT_MAX;
            for (int right = 0; right < n; right++) {
                sum += nums[right];                        // 右扩
                while (sum >= target) {                    // 满足 → 尝试缩短
                    minLen = min(minLen, right - left + 1);
                    sum -= nums[left];                     // 左缩
                    left++;
                }
            }
            return minLen == INT_MAX ? 0 : minLen;         // 没找到返 0
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minSubArrayLen(self, target: int, nums: list[int]) -> int:
            n = len(nums)
            left, total = 0, 0
            min_len = float('inf')
            for right in range(n):
                total += nums[right]
                while total >= target:
                    min_len = min(min_len, right - left + 1)
                    total -= nums[left]
                    left += 1
            return 0 if min_len == float('inf') else min_len
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} target
     * @param {number[]} nums
     * @return {number}
     */
    var minSubArrayLen = function(target, nums) {
        const n = nums.length;
        let left = 0, sum = 0, minLen = Infinity;
        for (let right = 0; right < n; right++) {
            sum += nums[right];
            while (sum >= target) {
                minLen = Math.min(minLen, right - left + 1);
                sum -= nums[left];
                left++;
            }
        }
        return minLen === Infinity ? 0 : minLen;
    };
    ```

## Complexity

- **Time**: O(n) — left, right 各自最多走 n 步, 摊销线性.
- **Space**: O(1).

## 相关题目

- [0027. Remove Element](../0027-remove-element/README.md) — 同向快慢双指针
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针
- [0727. Minimum Window Subsequence](../0727-minimum-window-subsequence/README.md) — 同向 + 回溯
- 0076\. Minimum Window Substring (待补) — 滑窗 + 哈希计数, "最短" 模板进阶
- 0003\. Longest Substring Without Repeating Characters (待补) — 滑窗 + 哈希, "最长" 模板
- 0904\. Fruit Into Baskets (待补) — 滑窗 + 至多 k 种
- 0713\. Subarray Product Less Than K (待补) — 滑窗 + 计数 (正数 → 单调)
- 0438\. Find All Anagrams in a String (待补) — 滑窗 + 字符计数
