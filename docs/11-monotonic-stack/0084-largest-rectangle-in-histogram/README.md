# 0084. Largest Rectangle in Histogram / 柱状图中最大的矩形

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Monotonic Stack, Array · 单调栈, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/largest-rectangle-in-histogram/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 柱状图每个柱子宽 1, 高 `heights[i]`. 求**能勾勒出的最大矩形面积** (矩形必须在柱状图轮廓内).

**中文**: 柱状图最大内接矩形面积.

## Key Insights

1. **🔑 核心思路: 以每根柱子为"最矮" 算最大矩形 / Treat each bar as the shortest**

    对每根柱子 `i`, 找左/右边**第一根比它矮**的位置 `L`, `R`. 那么以 `heights[i]` 为高的最大矩形宽度 = `R - L - 1`, 面积 = `heights[i] × (R - L - 1)`. 答案 = 所有柱子里这个面积的最大值.

    > 这是这题的**思维转换**: 不是穷举矩形, 而是"每个高度对应的最宽矩形". 转换后单调栈一遍 O(n) 搞定.

2. **🔑 单调递增栈 — 弹栈时同时确定左右边界 / Increasing stack: pop gives both sides at once**

    维护单调**递增** 栈 (栈底矮, 栈顶高). 来 `heights[i]`:

    - 若 `heights[i] < heights[stk.top()]` → 栈顶柱子找到了**右边第一个更矮 = i** !
    - 弹出栈顶 `top`. 此时**新栈顶就是 top 的左边第一个更矮**.
    - 算 `top` 为高的矩形: `width = i - new_top - 1`, `area = heights[top] × width`.
    - 重复至 `heights[i] >= stk.top()`. 把 `i` 压栈.

    > **一次弹栈同时拿到左右边界** 是这题最巧的地方. 跟 [0739](../0739-daily-temperatures/README.md) 单边模板不同, 这里栈帮你"同时" 锁两边.

3. **🔑 末尾补 `0` 哨兵: 强制清空栈 / Trailing 0 sentinel**

    Yang 用 `heights.push_back(0)` — 末尾加一个高度 0 的虚拟柱子. 它比任何正高度都小, 触发弹出栈里**剩下的所有** 柱子. 这些柱子的"右边第一个更矮" 就是这个虚拟 0.

    > 没有这步, 末尾递增的柱子留在栈里不被弹出, 它们的矩形没计算. 哨兵把"扫完后清栈" 的逻辑融入主循环.

4. **左边界为 `-1` (栈空) — 当前柱子最矮在到目前为止 / Empty stack → left boundary -1**

    Yang 写 `int left = stk.empty() ? -1 : stk.top();`. 栈空意味着前面没有比 top 更矮的 → 它能向左延伸到数组起点之前. **宽度 = `i - (-1) - 1 = i`**, 整数自洽.

    > 等价做法: 在栈底也放一个 -1 哨兵索引, 省掉这个 if. 两种写法都常见.

5. **为什么用严格 `<` 而非 `<=` / Strict less, not less-or-equal**

    `heights[i] < heights[stk.top()]` 严格. 等高的柱子**留在栈里**, 不弹出. 这**不影响正确性** — 等高的柱子在更晚被弹出时, 算出的宽度仍然正确 (那时右边界 `i` 是真正的第一个更矮).

    > 选 `<` 还是 `<=` 是这类题的常见选择. `<` 保证每个柱子只在它真正找到"右边严格更矮" 时才被结算, 结果一致.

6. **复杂度 O(n) 摊销 / Amortized O(n)**

    每个索引最多入栈一次, 出栈一次. **总 O(n)** — 是这题相对暴力 O(n²) 的极致优化.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int largestRectangleArea(vector<int>& heights) {
            heights.push_back(0);                                  // 末尾 0 哨兵, 强制清栈
            stack<int> stk;
            int maxArea = 0;
            for (int i = 0; i < (int)heights.size(); i++) {
                // 单调递增栈: 当前比栈顶矮 → 栈顶找到右边第一个更矮 = i
                while (!stk.empty() && heights[i] < heights[stk.top()]) {
                    int h = heights[stk.top()]; stk.pop();
                    int left = stk.empty() ? -1 : stk.top();       // 左边第一个更矮 (或 -1)
                    int w = i - left - 1;                          // 宽度: 严格在 (left, i) 之间
                    maxArea = max(maxArea, h * w);
                }
                stk.push(i);
            }
            return maxArea;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def largestRectangleArea(self, heights: list[int]) -> int:
            heights.append(0)                                      # 末尾哨兵
            stk = []
            max_area = 0
            for i, h_i in enumerate(heights):
                while stk and h_i < heights[stk[-1]]:
                    h = heights[stk.pop()]
                    # 栈空 → 左边界用 -1, 否则用新栈顶
                    left = stk[-1] if stk else -1
                    w = i - left - 1
                    max_area = max(max_area, h * w)
                stk.append(i)
            heights.pop()                                          # 还原输入 (副作用清理)
            return max_area
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} heights
     * @return {number}
     */
    var largestRectangleArea = function(heights) {
        heights.push(0);
        const stk = [];
        let maxArea = 0;
        for (let i = 0; i < heights.length; i++) {
            while (stk.length && heights[i] < heights[stk[stk.length - 1]]) {
                const h = heights[stk.pop()];
                const left = stk.length ? stk[stk.length - 1] : -1;
                const w = i - left - 1;
                if (h * w > maxArea) maxArea = h * w;
            }
            stk.push(i);
        }
        heights.pop();
        return maxArea;
    };
    ```

## Complexity

- **Time**: O(n) — 每索引入栈出栈各一次.
- **Space**: O(n) — 栈最坏存全部.

## 相关题目

- [0042. Trapping Rain Water](../0042-trapping-rain-water/README.md) — 同款单调栈思想, 但解法多
- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 单调栈最基础, 单向"找右边更大"
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 模板
- [0085. Maximal Rectangle](../0085-maximal-rectangle/README.md) — 二维, 把每行当 histogram 跑 0084
- 0907\. Sum of Subarray Minimums (待补) — 同款"以每元素为最小" + 单调栈
- 1856\. Maximum Subarray Min-Product (待补) — 同款思想, 跟前缀和组合
