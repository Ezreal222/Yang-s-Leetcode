# 0042. Trapping Rain Water / 接雨水

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Monotonic Stack, Two Pointers, DP · 单调栈, 双指针, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/trapping-rain-water/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给非负整数数组 `height` 表示柱子高度 (宽度为 1). 求**下雨后能接多少水**.

**中文**: 柱子高度数组, 求能接多少雨水.

## Key Insights

1. **🔑 两大解法: 单调栈 (横向分层) vs 双指针 (纵向分柱) / Two main approaches**

    | 解法 | 思路 | 空间 |
    |---|---|---|
    | **单调栈** (v1) | **横向分层** — 每弹一次栈算一个"凹槽" 的水 | O(n) 栈 |
    | **双指针** (v2 推荐) | **纵向分柱** — 每个位置上方能接多少水 = min(leftMax, rightMax) - height[i] | **O(1)** |

    > 同 O(n) 时间, 双指针 O(1) 空间 + 概念更清晰. **能写双指针就写双指针**.

2. **🔑 单调栈思路: 凹槽 = 左墙 + 底 + 右墙 / Stack: pop bottom, accumulate horizontal layer**

    维护**单调递减栈** (高的在底, 低的在顶). 来一个 `height[i]`:

    - 若 `height[i] > height[stk.top()]`, 栈顶 `bottom` 找到了右墙 `i`. 弹出 bottom.
    - 弹出后**新栈顶 left** 就是左墙. 水量 = `(min(height[left], height[i]) - height[bottom]) × (i - left - 1)`.
    - **若弹出 bottom 后栈空** → 没左墙, 不接水, break.
    - 重复直到 `height[i] ≤ stk.top()`. 把 `i` 压栈.

    > **横向一层一层算水**, 每弹一次栈算一层. 关键是"水的高度被两墙中较矮的那个限制".

3. **🔑 双指针思路: 单调维护 leftMax / rightMax / Two pointers: maintain max seen so far**

    两指针 `left`, `right` 从两端向中夹. 同时维护已扫过的最高墙 `leftMax`, `rightMax`. **谁矮谁先动**:

    - 若 `height[left] < height[right]`: **左侧水量被 leftMax 决定** (因为右侧已知存在 ≥ height[right] ≥ height[left] 的墙). 处理 `left`:
        - `height[left] ≥ leftMax` → 更新 `leftMax = height[left]` (新高度不接水)
        - 否则 → `water += leftMax - height[left]` (这一柱上方接水)
        - `left++`
    - 反之处理 `right`.

    **关键洞察**: 当 `height[left] < height[right]`, 我们**确定** 左柱上方接的水只看 `leftMax` (因为 rightMax ≥ height[right] > height[left] ≥ leftMax, 即使后面 leftMax 增长仍 ≤ rightMax). 不需要知道 rightMax 具体值.

    > 双指针的精妙在于"**因为对面更高, 这边的水高度已被锁定**" 这个不变式. 看一次就懂.

4. **`if (stk.empty()) break;` — 栈解的边界 / Stack edge case**

    弹出 bottom 后若栈空 → 没有左墙, 该 bottom 上方接不到水, 直接 break 退出 while. 漏写则下面 `stk.top()` UB.

5. **复杂度 O(n) 两版都是 / Both O(n)**

    单调栈每索引入出各一次. 双指针 left/right 总移动 n 次. **都是线性**.

6. **第三种解法: 预处理 leftMax / rightMax 数组 (DP 思路) / Third way: precompute arrays**

    对每个 i 预算 `leftMax[i]` 和 `rightMax[i]`, 答案 = `Σ min(leftMax[i], rightMax[i]) - height[i]`. **O(n) 时间, O(n) 空间** — 不如双指针省, 但概念上是双指针的"无压缩版", 入门理解友好.

    ```cpp
    // O(n²) 朴素 → O(n) DP → O(1) 双指针, 是一条优化链
    ```

## Solution

=== "C++"
    === "v1: 单调栈 (横向分层)"
        ```cpp
        class Solution {
        public:
            int trap(vector<int>& height) {
                int n = height.size(), water = 0;
                stack<int> stk;                                    // 栈存索引, 维护单调递减
                for (int i = 0; i < n; i++) {
                    while (!stk.empty() && height[i] > height[stk.top()]) {
                        int bottom = stk.top(); stk.pop();
                        if (stk.empty()) break;                    // 没左墙, 不接水
                        int left = stk.top();
                        int h = min(height[left], height[i]) - height[bottom];
                        int w = i - left - 1;
                        water += h * w;
                    }
                    stk.push(i);
                }
                return water;
            }
        };
        ```

    === "v2 推荐: 双指针 O(1) 空间"
        ```cpp
        class Solution {
        public:
            int trap(vector<int>& height) {
                int left = 0, right = height.size() - 1;
                int leftMax = 0, rightMax = 0, water = 0;
                while (left < right) {
                    if (height[left] < height[right]) {
                        // 左边矮 → 水被 leftMax 锁定 (右边对面更高, 确保不漏)
                        height[left] >= leftMax ? leftMax = height[left]
                                                : water += leftMax - height[left];
                        left++;
                    } else {
                        height[right] >= rightMax ? rightMax = height[right]
                                                  : water += rightMax - height[right];
                        right--;
                    }
                }
                return water;
            }
        };
        ```

    === "v3: 预处理 leftMax/rightMax (O(n) 空间)"
        ```cpp
        class Solution {
        public:
            int trap(vector<int>& height) {
                int n = height.size();
                vector<int> lmax(n), rmax(n);
                lmax[0] = height[0];
                for (int i = 1; i < n; i++) lmax[i] = max(lmax[i - 1], height[i]);
                rmax[n - 1] = height[n - 1];
                for (int i = n - 2; i >= 0; i--) rmax[i] = max(rmax[i + 1], height[i]);
                int water = 0;
                for (int i = 0; i < n; i++) water += min(lmax[i], rmax[i]) - height[i];
                return water;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def trap(self, height: list[int]) -> int:
            # v2 双指针 O(1) — 跟 C++ v2 一一对应
            left, right = 0, len(height) - 1
            left_max = right_max = water = 0
            while left < right:
                if height[left] < height[right]:
                    # 左矮 → 水量看 leftMax, 右边对面更高保底
                    if height[left] >= left_max:
                        left_max = height[left]
                    else:
                        water += left_max - height[left]
                    left += 1
                else:
                    if height[right] >= right_max:
                        right_max = height[right]
                    else:
                        water += right_max - height[right]
                    right -= 1
            return water
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} height
     * @return {number}
     */
    var trap = function(height) {
        let left = 0, right = height.length - 1;
        let leftMax = 0, rightMax = 0, water = 0;
        while (left < right) {
            if (height[left] < height[right]) {
                if (height[left] >= leftMax) leftMax = height[left];
                else                          water += leftMax - height[left];
                left++;
            } else {
                if (height[right] >= rightMax) rightMax = height[right];
                else                            water += rightMax - height[right];
                right--;
            }
        }
        return water;
    };
    ```

## Complexity

- **Time**: O(n) — 三种解法都是.
- **Space**: O(n) (v1 栈 / v3 数组) / **O(1)** (v2 双指针).

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 单调栈基础
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — 单调栈最小模板
- 0011\. Container With Most Water (待补) — 同款双指针 + "谁矮谁动" 思想
- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 单调栈经典进阶, "一次弹栈同时拿左右边界"
- 0407\. Trapping Rain Water II (待补) — 二维版, 用最小堆
- 0238\. Product of Array Except Self (待补) — 同款"前缀 + 后缀" 预处理思想 (跟 v3 相似)
