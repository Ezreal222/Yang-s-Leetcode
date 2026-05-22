# 0280. Wiggle Sort / 摆动排序

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Array, Sort · 贪心, 数组, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/wiggle-sort/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: In-place reorder `nums` so `nums[0] <= nums[1] >= nums[2] <= nums[3] ...` (valley, peak, valley, peak, …). Any valid arrangement works.

**中文**: 原地把 `nums` 排成 `nums[0] <= nums[1] >= nums[2] <= nums[3] ...` (谷, 峰, 谷, 峰 …). 任意一组合法答案都行.

## Key Insights

1. **一遍扫 + 相邻对修正 / Single pass, fix-by-swap**

    位置 `i` 的奇偶性决定它**该**是谷还是峰:

    - `i` 偶 (该是谷): 若 `nums[i] > nums[i+1]` → swap (让 `i` 更小).
    - `i` 奇 (该是峰): 若 `nums[i] < nums[i+1]` → swap (让 `i` 更大).

    一句话: **看一对相邻, 违反就交换**. 不用 sort, O(n).

2. **交换后为什么不会破坏前一对 / Swap-safety lemma**

    关键证明: 修正第 `i` 对后, 第 `i-1` 对仍然合法.

    例: `i` 偶 (谷), `nums[i] > nums[i+1]`, 要 swap.

    - swap 前 `i-1` 对已合法: `nums[i-1] >= nums[i]` (i-1 是峰).
    - swap 后 `nums[i]` 变成原来的 `nums[i+1]` — 更小.
    - 因为 `nums[i+1] < nums[i] <= nums[i-1]` → swap 后仍有 `nums[i-1] >= nums[i]` ✓.

    奇位 (峰) 同理 (符号反过来). 所以**只往前扫一遍即可**, 不需要回头修.

3. **跟 [0376 Wiggle Subsequence](../0376-wiggle-subsequence/README.md) 区别 / vs 0376**

    | 题 | 输入 | 操作 | 求什么 |
    |---|---|---|---|
    | **0280 (本题)** | 任意数组 | **原地交换 / 修改** | 任一合法摆动排列 |
    | 0376 | 任意数组 | **不改, 只数** | 最长摆动子序列长度 |

    0376 是"只看, 数转折"; 0280 是"动手, 强制变成摆动". 两个完全不同的题, 共享"摆动" 这个概念.

4. **vs 标准做法 (sort + 间隔填) / Compare to sort-based approach**

    经典做法: 整体 sort, 然后**先填偶位再填奇位** (`[1,2,3,4,5]` → `[1,3,2,5,4]`). O(n log n).

    本题贪心**O(n)** 更快, 且不需要额外空间. 一句话: **只要邻接对满足, 全局就是摆动** — 不需要全局 sort.

5. **vs 0324 Wiggle Sort II (待补) / Stricter cousin**

    0324 要求**严格** `<` 和 `>` (不能 `=`). 本题允许 `<=` 和 `>=`. 严格版用本题贪心**不一定能解**, 要先 sort + 间隔填 + 中位数处理 (相等元素扎堆是难点).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        void wiggleSort(vector<int>& nums) {
            for (int i = 0; i + 1 < (int)nums.size(); i++) {       // 写 i+1<size 避免 nums.size()-1 在空数组时溢出
                if (i % 2 == 0) {                                  // 偶位 (谷) 应 ≤ 后一位
                    if (nums[i] > nums[i + 1]) swap(nums[i], nums[i + 1]);
                } else {                                           // 奇位 (峰) 应 ≥ 后一位
                    if (nums[i] < nums[i + 1]) swap(nums[i], nums[i + 1]);
                }
            }
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def wiggleSort(self, nums: list[int]) -> None:
            # 一行版本: 用 (i % 2 == 0) 这个 bool 来决定关系符方向
            # nums[i], nums[i+1] = nums[i+1], nums[i] 是 Python 优雅的 swap (元组解包, 无需 temp)
            for i in range(len(nums) - 1):
                if (i % 2 == 0 and nums[i] > nums[i + 1]) or \
                   (i % 2 == 1 and nums[i] < nums[i + 1]):
                    nums[i], nums[i + 1] = nums[i + 1], nums[i]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {void} 原地修改
     */
    var wiggleSort = function(nums) {
        for (let i = 0; i < nums.length - 1; i++) {
            // 解构赋值 [a, b] = [b, a] 是 ES6 优雅 swap, 等价 Python 元组解包
            if (i % 2 === 0) {
                if (nums[i] > nums[i + 1]) [nums[i], nums[i + 1]] = [nums[i + 1], nums[i]];
            } else {
                if (nums[i] < nums[i + 1]) [nums[i], nums[i + 1]] = [nums[i + 1], nums[i]];
            }
        }
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(1) — 原地交换.

## 相关题目

- [0376. Wiggle Subsequence](../0376-wiggle-subsequence/README.md) — 同款"摆动" 概念, 但是数子序列长度
- 0324\. Wiggle Sort II (待补) — 严格 `<` `>` 版, 贪心失效, 要 sort + 间隔填
- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同款"一遍扫 + 局部修正" 贪心入门
- 0581\. Shortest Unsorted Continuous Subarray (待补) — 同款"扫一遍找违规位"
- 0665\. Non-decreasing Array (待补) — 同款"邻接对违规修复" 思路
