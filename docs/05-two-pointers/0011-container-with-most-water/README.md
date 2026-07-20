# 0011. Container With Most Water / 盛最多水的容器

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, Greedy, Array · 双指针, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/container-with-most-water/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Max area between two lines** → **opposite two pointers + greedy**: area = `(r-l) × min(h[l], h[r])`; **always move the shorter side inward** (moving the taller side can't help — width shrinks, height still capped by the same short side).
>
> **中文**: **两线间最大蓄水面积** → **对撞双指针 + 贪心**: 面积 = `(r-l) × min(h[l], h[r])`; **每次移动短边** (动长边只会宽变小 + 高度被同一短边压住 → 面积必减).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `height` 表示每个位置的立柱高度. 选两根柱子, **中间容水最多**. 求最大面积.

- 面积 = **宽 (下标差) × 高 (min(两柱))**.

**中文**: 两柱之间最多能装多少水.

## Key Insights

1. **🔑 朴素 O(n²) → 对撞双指针 O(n) / Brute → Greedy two pointers**

    朴素: 枚举所有 (i, j), n² 对, TLE.

    **对撞**: `left = 0, right = n-1`. 每步算面积后**贪心**移动**短边**. 一遍扫 O(n).

2. **🔑 灵魂证明: 为啥可以只移短边? / Why moving only the shorter side is safe**

    假设 `h[l] < h[r]`. 考虑**所有以 l 为左端点** 的容器 (l 配 l+1, l+2, ..., r):

    - **宽度**: r - l 最大, 其他都 ≤ 它.
    - **高度**: min(h[l], h[k]) ≤ h[l]. 即使 h[k] 很高, 也被 h[l] **压顶**.
    - → 这类容器**最大面积**就是当前 (l, r).

    → **"l 这个端点" 已经玩完了** — 移动 l (短边) 到下一个位置. **移动 r (长边)** 只会宽变小, 高度仍受 h[l] 限制 → 面积必减.

    > **贪心正确性 = 每一步"排除一个端点的所有可能"**. 双指针相向而行 → n 步 O(n).

3. **🔑 短边 == 长边时随便 / Tie: move either**

    若 `h[l] == h[r]`, 移哪个都对 (两侧都"玩完"). Yang 的 `if (h[l] < h[r]) l++; else r--;` 在等值时移 r — 无所谓, 结果一致.

4. **🔑 每步都算面积 → 遍历中记 max / Track max as we go**

    - 每步算 `area = (r - l) × min(h[l], h[r])`.
    - `maxA = max(maxA, area)`.
    - 最后返 maxA.

    > **"贪心走 + 顺路记极值"** 是 O(n) 求最优的通用套路.

5. **🔑 跟 [0042 Trapping Rain Water](../../11-monotonic-stack/0042-trapping-rain-water/README.md) 的关系 / vs 0042**

    | | **0011 (本题)** | 0042 Trapping Rain Water |
    |---|---|---|
    | 问 | **两柱间**最多容水 | **所有柱** 各自能蓄多少水 |
    | 解 | 对撞 + 贪心 O(n) | 对撞 + 维护双向 max O(n) |
    | 输出 | 一个最大面积 | 总蓄水量 |

    > **同一"对撞双指针"** 但状态不同. 0011 只需高度对比, 0042 要维护"目前两侧的最高柱" (前缀/后缀最大).

6. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    - Time: 双指针相向而行, 最多 n-1 步.
    - Space: 两指针 + 一 max.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxArea(vector<int>& height) {
            int left = 0, right = (int)height.size() - 1;
            int maxA = 0;
            while (left < right) {
                int area = (right - left) * min(height[left], height[right]);
                maxA = max(maxA, area);
                if (height[left] < height[right]) left++;        // 移短边
                else right--;
            }
            return maxA;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxArea(self, height: list[int]) -> int:
            left, right = 0, len(height) - 1
            max_a = 0
            while left < right:
                # min() + 乘法一行算面积. max() 记录; 都是内建, 常数微小
                area = (right - left) * min(height[left], height[right])
                if area > max_a: max_a = area
                # 贪心移短边. 等值时移哪都行, 这里默认 else 移 right
                if height[left] < height[right]:
                    left += 1
                else:
                    right -= 1
            return max_a
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} height
     * @return {number}
     */
    var maxArea = function(height) {
        let left = 0, right = height.length - 1;
        let maxA = 0;
        while (left < right) {
            // Math.min / Math.max — JS 无原生 min/max 运算符, 全靠函数
            const area = (right - left) * Math.min(height[left], height[right]);
            maxA = Math.max(maxA, area);
            if (height[left] < height[right]) left++;
            else right--;
        }
        return maxA;
    };
    ```

## Complexity

- **Time**: O(n) — 双指针一遍相向.
- **Space**: O(1) — 两指针 + 一 max.

## 相关题目

- [0344. Reverse String](../0344-reverse-string/README.md) — 对撞最简
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞合并
- [0125. Valid Palindrome](../0125-valid-palindrome/README.md) — 对撞 + 跳非法
- [0015. 3Sum](../0015-3sum/README.md) — 对撞 + 3 层去重
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 不定长滑窗
- [0042. Trapping Rain Water](../../11-monotonic-stack/0042-trapping-rain-water/README.md) — 对撞 + 双向 max, "所有柱" 蓄水
- [0084. Largest Rectangle in Histogram](../../11-monotonic-stack/0084-largest-rectangle-in-histogram/README.md) — 单调栈, 最大矩形
- [0085. Maximal Rectangle](../../11-monotonic-stack/0085-maximal-rectangle/README.md) — 二维版
