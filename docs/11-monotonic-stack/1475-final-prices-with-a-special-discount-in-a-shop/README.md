# 1475. Final Prices With a Special Discount in a Shop / 商品折扣后的最终价格

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Monotonic Stack, Array · 单调栈, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 商品价格 `prices`. 买第 `i` 件可享受折扣 = 数组中**之后第一个 `≤ prices[i]`** 的价格 (即 `prices[j]`, `j > i` 最小). 没有则不打折. 返回每件商品的**最终价格**.

**中文**: 第 i 件可减 = 之后第一个 ≤ 当前价的值, 没有不减. 求最终价格.

## Key Insights

1. **🔑 "下一个更小或等" (NSE) — NGE 的镜像 / Next Smaller-or-Equal**

    跟 [0739 / 0496 / 0503](../0739-daily-temperatures/README.md) 的"下一个更大" 镜像. 把 `>` 换成 `<=` 就是 NSE 模板.

    | 模板 | 比较 | 栈方向 |
    |---|---|---|
    | NGE (下一个更大) | `nums[i] > stk.top()` | 单调递减栈 |
    | **NSE (下一个更小或等, 本题)** | **`nums[i] <= stk.top()`** | **单调递增栈** |

    > **`<` 还是 `<=`** 看题目要求"严格" 还是"允许等". 本题题目"≤", 用 `<=`.

2. **🔑 原地修改: prices 直接作为答案 / In-place modification**

    Yang 的优雅做法: **不分配新数组**, 直接 `prices[stk.top()] -= prices[i]`. 弹栈时把折扣减进原价 — 因为弹栈意味着"找到了它的下一个 ≤", 现在就是结算的时候.

    最终栈里剩的元素没有找到下一个 ≤ → 它们的 prices 没被改, 原价保留 (即没折扣).

    > **原地改 + 栈存索引** 是这题最简洁的写法. O(1) 额外空间 (栈不算).

3. **状态: 栈存索引, 维持单调递增 / Increasing stack of indices**

    栈中索引对应的价格**单调递增** (栈底小栈顶大). 来 `prices[i]`:

    - 若 `prices[i] <= prices[stk.top()]` → 栈顶找到了它的 NSE = `prices[i]`. 减去这个折扣, 弹出.
    - 重复直到栈空或栈顶 `<` `prices[i]`. push `i`.

4. **复杂度 O(n) 摊销 / Amortized O(n)**

    每索引入栈出栈各一次. 跟所有单调栈题同套路.

5. **跟 [0739](../0739-daily-temperatures/README.md) 对比 / vs 0739**

    | | 0739 (NGE 距离) | 1475 (NSE 折扣) |
    |---|---|---|
    | 找什么 | 下一个**更大** | 下一个**更小或等** |
    | 栈方向 | 单调递减 | 单调递增 |
    | 输出 | `res[idx] = i - idx` (距离) | `prices[idx] -= prices[i]` (扣折) |
    | 比较 | `>` 严格 | `<=` 允许等 |

    > 单调栈所有变种就是这四列组合: **找什么 / 栈方向 / 输出形式 / 严格性**. 记住这四维就能套出来.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> finalPrices(vector<int>& prices) {
            stack<int> stk;                                        // 栈存索引
            for (int i = 0; i < (int)prices.size(); i++) {
                // 栈顶 ≥ 当前 → 栈顶找到 NSE = prices[i], 扣折
                while (!stk.empty() && prices[i] <= prices[stk.top()]) {
                    prices[stk.top()] -= prices[i];                // 原地改, 省额外空间
                    stk.pop();
                }
                stk.push(i);
            }
            return prices;                                         // 没被改的 = 没折扣
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def finalPrices(self, prices: list[int]) -> list[int]:
            stk = []
            for i, p in enumerate(prices):
                # 栈顶 ≥ 当前 → 扣折
                while stk and prices[stk[-1]] >= p:
                    prices[stk.pop()] -= p
                stk.append(i)
            return prices
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} prices
     * @return {number[]}
     */
    var finalPrices = function(prices) {
        const stk = [];
        for (let i = 0; i < prices.length; i++) {
            while (stk.length && prices[i] <= prices[stk[stk.length - 1]]) {
                prices[stk.pop()] -= prices[i];
            }
            stk.push(i);
        }
        return prices;
    };
    ```

## Complexity

- **Time**: O(n) — 每索引入栈出栈各一次.
- **Space**: O(n) (栈) / **O(1)** 额外 (原地改 prices).

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — NGE 母题, 求距离
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 基础
- [0503. Next Greater Element II](../0503-next-greater-element-ii/README.md) — 循环数组 NGE
- [0901. Online Stock Span](../0901-online-stock-span/README.md) — 流式版单调栈
- 0496\. (重复)
- 0856\. Score of Parentheses — 同款"栈做计数操作", 见 [§06 0856](../../06-stack-queue/0856-score-of-parentheses/README.md)
- 0907\. Sum of Subarray Minimums (待补) — 同款 NSE + 贡献法
