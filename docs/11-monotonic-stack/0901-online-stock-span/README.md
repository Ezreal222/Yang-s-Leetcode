# 0901. Online Stock Span / 股票价格跨度

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Streaming, Design · 单调栈, 流式, 设计
    - **Link**: [LeetCode](https://leetcode.com/problems/online-stock-span/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 实现 `StockSpanner.next(price)`: 返回**当前价 (含今天) 向前连续 `≤` 当前价的天数** (即"今天之前有多少天价格 ≤ 今天").

**中文**: 每次 `next(price)` 返回从今天向前连续 ≤ 当前价的天数 (含今天).

## Key Insights

1. **🔑 流式版单调栈 — 数据一次一个进入, 不知道未来 / Online/streaming monotonic stack**

    跟 [0739](../0739-daily-temperatures/README.md) / [0496](../0496-next-greater-element-i/README.md) 离线整数组不同, 本题是**流式**: 每次 `next` 来一个新价格, 立刻回答. 单调栈天然支持这种模式 — 只看历史, 不需要未来.

    > **单调栈的天然属性: 在线友好**. 0901 把这个属性显式化.

2. **🔑 栈存 `(price, span)` 对 — 不存索引/天数 / Store (price, span), not index**

    跟 0739 (栈存索引算距离) 不同, 这里栈存 `(price, span)` — `span` 是这个价格"吸收" 了多少天.

    新价 `p` 来时, 弹出所有 `top.price ≤ p` 的元素, **累加它们的 span 进当前 res**. 然后把 `(p, res)` 压栈.

    > **栈存"块" 而不是"单点"** 是流式题的常见技巧 — 弹出时块的总跨度一次性合并, 不用逐个回溯.

3. **状态: 栈中元素的 price 严格递减 / Strictly decreasing stack of prices**

    维持递减不变量: 一旦新价 `≥` 栈顶, 栈顶被"吃掉"; 等于也吃 (因为题目 `span` 算 ≤ 当前价).

    `>=` 不是 `>` — 等价价的天数应该被吸收, 否则会漏算.

4. **`res = 1` 起步: 今天自己算一天 / Start with self-day**

    `res = 1` 初值表示"今天本身算 1 天". 之后每弹一次累加被吃天数. 漏写 → res 总比真实答案少 1.

5. **🔑 摊销 O(1) per call / Amortized O(1)**

    单次 `next` 看似 O(k) (k = 被弹元素数), 但**每个元素一生只入栈一次出栈一次**. 跨 n 次 `next` 总操作 O(n) → **平均 O(1)** per call. 跟 0739/0084 同款摊销论证.

    > **流式 + 摊销 O(1)** 是这题面试中的卖点 — 比"每次重扫历史" 的 O(n) per call 快 n 倍.

## Solution

=== "C++"
    ```cpp
    class StockSpanner {
    public:
        stack<pair<int, int>> stk;                                 // (price, span)
        StockSpanner() {}

        int next(int price) {
            int res = 1;                                           // 今天自己算 1 天
            // 弹出所有 ≤ 当前价的栈顶, 累加它们的 span
            while (!stk.empty() && price >= stk.top().first) {
                res += stk.top().second;
                stk.pop();
            }
            stk.push({price, res});                                // 压入 (今天价, 累计天数)
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class StockSpanner:
        def __init__(self):
            # 栈存 (price, span) 元组, 跟 C++ pair 一一对应
            self.stk: list[tuple[int, int]] = []

        def next(self, price: int) -> int:
            res = 1
            # 弹出 price ≤ 当前价的, 累加跨度
            while self.stk and self.stk[-1][0] <= price:
                res += self.stk.pop()[1]
            self.stk.append((price, res))
            return res
    ```

=== "JavaScript"
    ```javascript
    var StockSpanner = function() {
        this.stk = [];                                             // [price, span]
    };

    /**
     * @param {number} price
     * @return {number}
     */
    StockSpanner.prototype.next = function(price) {
        let res = 1;
        while (this.stk.length && this.stk[this.stk.length - 1][0] <= price) {
            res += this.stk.pop()[1];
        }
        this.stk.push([price, res]);
        return res;
    };
    ```

## Complexity

- **Time**: 摊销 O(1) per `next`, 总 O(n) across n calls.
- **Space**: O(n) — 栈最坏存所有元素 (严格递减序列).

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 离线版"找右边第一个更大", 同栈思想
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 模板
- [0503. Next Greater Element II](../0503-next-greater-element-ii/README.md) — 循环数组版
- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 单调栈经典 Hard
- 0496\. (重复) Next Greater Element II — already linked
- [1019. Next Greater Node In Linked List](../1019-next-greater-node-in-linked-list/README.md) — 链表上的 NGE, 同款单调栈
